
# Installing and Configuring the Network Server (NS) using ChirpStack (Debian/Ubuntu)

In our set up, we have a dedicated NS Virtual Private Server which is accessible externally by distinct physical assress. 

This page covers setting up the NS which is detailed in different sections as follows:

 * [NS Setup]
 * [Verifying Communication between RGW->NS]

## NS Setup

###	Prerequisites

1.	A VM or Physical instance installed with Debian or Ubuntu accessible via the Internet
2.	MQTT Broker (e.g. Mosquitto)
3.	Persistent Database (e.g. PostgreSQL)
4.	Non-persistent Database (e.g. Redis)

###	Installing the Prerequisites

```sh
sudo apt-get install mosquitto postgresql redis-server
```

### Setting up the Prerequisites

Setup your PostgreSQL database for your ChirpStack NS

```sh
$ sudo -u postgres psql
> create role chirpstack_ns with login password 'dbnspassword';
> create database chirpstack_ns with owner chirpstack_ns;
> \q
```

Verify that you are able to connect to the PSQL database

```sh
$ psql -h localhost -U chirpstack_ns -W chirpstack_ns
```

###	Obtaining and Installing the ChirpStack NS

Download the [binary] for ChirpStack NS.

Setup the NS, either using the binary from the link above, or by using Debian package manager. To install from the package manager, you will also need ```apt-transport-https``` to connect to the repository.

```sh
$ sudo apt install apt-transport-https
$ sudo apt-get install dirmngr --install-recommends
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1CE2AFD36DBCCA00
$ sudo echo "deb https://artifacts.chirpstack.io/packages/3.x/deb stable main" | sudo tee /etc/apt/sources.list.d/chirpstack.list
$ sudo apt-get update
$ sudo apt-get install chirpstack-network-server
```

### Configuration for the ChirpStack NS
The configuration file is located in the chirpstack-network-server folder in /etc. It is a “toml“ file.
```sh
$ /etc/chirpstack-network-server/chirpstack-network-server.toml
```

ChirpStack recommends checking to following sections and modifying the parameters (if required) in the above .toml file when setting up a ChirpStack NS:
 * 1. [postgresql]
 * 2. [redis]
 * 3. [network_server]
 * 4. [metrics]

#### ```postgresql``` section

Update the ```dsn``` parameter with the parameters you provided when setting up your own PostgreSQL database:
```sh
dsn="postgres://chirpstack_ns:dbnspassword@localhost/chirpstack_ns?sslmode=disable"
``` 

The following parameter in the default configuration file is “postgresql.automigrate” which is useful when upgrading ChirpStack. Set it as you wish :

```sh
automigrate=true
```

#### ```redis``` section

If you changed the default port for redis or if you host redis on a different machine, don’t forget to change the ```redis.url``` parameter


####  ```network_server``` section

You may set your LoRaWAN NetID (if you have one).  The net_id parameter may be set to “000000” or “000001” for experimental network. But one need a NetID to do passive roaming:

```sh
.
net_id=123456
```
Also check out if your LoRaWAN transmission band is consistent with the regulation for the region you live in (As we are located in Europe and use the 868MHz band, we set it in the configuration (other possible values are indicated in the default configuration file)
```sh
[network_server]
.
.
   [network_server.band]
   .
   .
   name=”EU_863”
```

####  ```metrics``` section

You can use either ```local``` for the system local time zone or use settings such as ```“Europe/Paris”```


### 	Starting the ChirpStack NS
To (re)start and stop ChirpStack NS depends on if your distribution uses “init.d” or “systemd”:

init.d
```sh
sudo /etc/init.d/chirpstack-network-server [start|stop|restart|status]
```
systemd
```sh
sudo systemctl [start|stop|restart|status] chirpstack-network-server
```

### 	Post Sanity Check for the ChirpStack NS

init.d
```sh
tail -f /var/log/chirpstack-network-server/chirpstack-network-server.log
```
systemd
```sh
journalctl -u chirpstack-network-server -f -n 50
```
A successful start of the NS will have the following Output in the log file

```sh
Jul 15 12:00:25 vps323914 systemd[1]: Started ChirpStack Network Server.
Jul 15 12:00:25 vps323914 chirpstack-network-server[5420]: time="2020-07-15T12:00:25+02:00" level=info msg="starting ChirpStack Network Server" band=EU868 docs="https://www.chirpstack.io/" net_id=123456 version=3.9.0
Jul 15 12:00:25 vps323914 chirpstack-network-server[5420]: time="2020-07-15T12:00:25+02:00" level=info msg="storage: setting up storage module"
Jul 15 12:00:25 vps323914 chirpstack-network-server[5420]: time="2020-07-15T12:00:25+02:00" level=info msg="storage: setting up Redis client"
Jul 15 12:00:25 vps323914 chirpstack-network-server[5420]: time="2020-07-15T12:00:25+02:00" level=info msg="storage: connecting to PostgreSQL"
Jul 15 12:00:25 vps323914 chirpstack-network-server[5420]: time="2020-07-15T12:00:25+02:00" level=info msg="storage: applying PostgreSQL data migrations"
Jul 15 12:00:25 vps323914 chirpstack-network-server[5420]: time="2020-07-15T12:00:25+02:00" level=info msg="storage: PostgreSQL data migrations applied" count=25
Jul 15 12:00:25 vps323914 chirpstack-network-server[5420]: time="2020-07-15T12:00:25+02:00" level=info msg="gateway/mqtt: connecting to mqtt broker" server="tcp://localhost:1883"
...
...
```

*For now, if you followed all the above indications. You should have a running ChirpStack stack Network Server* 


## Post Sanity Check from RGW->NS Setup

The objective in this section is to verify the communication between the RGW->NS (2) as shown in the figure:

<p align="center">
  <img width="760" height="300" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig14.png?raw=true">
</p>

### Verify whether the NS receives data from the RGW

Depending on your OS, one of the following commands will show you the logs:

```sh
journalctl -f -n 100 -u chirpstack-network-server
tail -f -n 100 /var/log/chirpstack-network-server/chirpstack-network-server.log
```
```diff
+ Expected log output
```

```sh
INFO[0163] backend/gateway: uplink frame received
INFO[0164] device gateway rx-info meta-data saved        dev_eui=0101010101010101
INFO[0164] device-session saved                          dev_addr=018f5aa9 dev_eui=0101010101010101
INFO[0164] finished client unary call                    grpc.code=OK grpc.method=HandleUplinkData grpc.service=as.ApplicationServerService grpc.time_ms=52.204 span.kind=client system=grpc
```

```diff
- But the NS logs gives an error
```

```sh
Jul 16 11:11:01 vps323914 chirpstack-network-server[21156]: time="2020-07-16T11:11:01+02:00" level=info msg="gateway/mqtt: uplink frame received" gateway_id=00800000a0000825 uplink_id=634ac2e8-939f-4af8-8ac4-3407bae732ba
Jul 16 11:11:01 vps323914 chirpstack-network-server[21156]: time="2020-07-16T11:11:01+02:00" level=info msg="uplink: frame(s) collected" ctx_id=b6a1e382-ae69-49b4-b3ff-20c5dacaee21 mtype=JoinRequest uplink_ids="[634ac2e8-939f-4af8-8ac4-3407bae732ba]"
Jul 16 11:11:01 vps323914 chirpstack-network-server[21156]: time="2020-07-16T11:11:01+02:00" level=error msg="get gateway error" ctx_id=b6a1e382-ae69-49b4-b3ff-20c5dacaee21 error="get gateway error: object does not exist" gateway_id=00800000a0000825
Jul 16 11:11:01 vps323914 chirpstack-network-server[21156]: time="2020-07-16T11:11:01+02:00" level=error msg="uplink: processing uplink frame error" ctx_id=b6a1e382-ae69-49b4-b3ff-20c5dacaee21 error="get device error: object does not exist"
```

In this log, the NS has processed the uplink frame from the RGW, and emits an error message mentioning that the object (ED) is not part of the NS known objects (EDs)

To make sure that the NS understands the data from the ED, the ED should be activated to interface with the NS. As per the [LoRaWAN Backend Specifications], there are two ways that the ED could be activated : 
 * 1. Activation-by-Personalization (ABP) activated EDs
 * 2. Over-the-Air Activated (OTAA) EDs

```diff
+ Our focus will be on OTAA EDs
```
To activate the ED via OTAA, NS uses the Join Server (JS) as per the [LoRaWAN Backend Specifications]. To complete the verification of the RGW->NS interface one has to validate the AS Post Sanity (in our set up JS is part of the AS) Check as per our set up.

## Pointer Section

In the event of one using NS that is not Chirpstack, please make sure that you [Verifying Communication between RGW->NS] . Once verified, Next section to follow : [AS_Setup]

If you are using Chirpstack NS, Next section to follow : [AS_Setup]

 * In any case, If you want to go back to the [Readme Page]
 * In any case, If you want to go back to the [Architecture page]
 * In any case, If you want to go back to the [Setting up the End-Device]
 * In any case, If you want to go back to the [Setting up the GW]

[NS Setup]: #ns-setup
[Verifying Communication between RGW->NS]: #post-sanity-check-from-rgw-ns-setup
[binary]:  https://www.chirpstack.io/network-server/overview/downloads/
[postgresql]: #postgresql-section
[redis]: #redis-section
[network_server]: #network_server-section
[metrics]: #metrics-section
[LoRaWAN Backend Specifications]: https://lora-alliance.org/resource-hub/lorawanr-back-end-interfaces-v10
[AS_Setup]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/ApplicationServer-Setup.md
[Setting up the GW]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Gateway-Setup.md
[Architecture page]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Architecture.md
[Readme Page]: https://github.com/sandoche2k/IoTRoam-Tutorial
[Setting up the End-Device]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/End-Device.md





