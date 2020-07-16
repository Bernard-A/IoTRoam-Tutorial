
# Installing and Configuring the Network Server (NS) and Application Server (AS) using ChirpStack (Debian/Ubuntu)

In our set up, we have a dedicated NS Virtual Private Server and an AS Virtual Private Server which is accessible externally by distinct physical assress. 

This page covers setting up the NS and AS which are detailed in different sections as follows:

 * [NS Setup]
 * [As Setup]
 * [Verifying Communication between RGW->NS->AS]

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

##	AS Setup

### Prerequisites

1.	A VM or Physical instance installed with Debian or Ubuntu accessible via the Internet
2.	MQTT Broker (e.g. Mosquitto)
3.	Persistent Database (e.g. PostgreSQL)
4.	Non-persistent Database (e.g. Redis)

### Installing the Prerequisites
```sh
sudo apt-get install mosquitto postgresql redis-server
```

###	Setting up the Prerequisites

Setup your PostgreSQL database for your ChirpStack AS
```sh
$ sudo -u postgres psql
> create role chirpstack_as with login password 'dbaspassword';
> create database chirpstack_as with owner chirpstack_as;
> \c chirpstack_as
> create extension pg_trgm;  ## If you get error install "sudo apt install postgresql-contrib"
> create extension hstore;
> \q
```

Verify the set up
```sh
$ psql -h localhost -U chirpstack_as -W chirpstack_as
```

###	Obtaining and Installing the ChirpStack AS

Download the [AS binary].

Setup the As, either using the binary from the link above, or by using Debian package manager. To install from the package manager, you will also need ```apt-transport-https``` to connect to the repository

```sh
$ sudo apt install apt-transport-https
$ sudo apt-get install dirmngr --install-recommends
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1CE2AFD36DBCCA00
$ sudo echo "deb https://artifacts.chirpstack.io/packages/3.x/deb stable main" | sudo tee /etc/apt/sources.list.d/chirpstack.list
$ sudo apt-get update
$ sudo apt-get install chirpstack-application-server
```

###	Configuration for the ChirpStack AS
The configuration file is located as follows:

```sh
$ /etc/chirpstack-application-server/chirpstack-application-server.toml
```
ChirpStack recommends checking to following parameters in the above .toml file when setting up a ChirpStack AS:

* 1. [AS postgresql]
* 2. [AS redis]
* 3. [AS application_server.external_api]

#### AS ```postgresql``` section

Update the “dsn” parameter with the parameters you provided when setting up your own PostgreSQL database:

```sh
dsn="postgres://chirpstack_as:dbaspassword@localhost/chirpstack_as?sslmode=disable"
```

The following parameter in the default configuration file is “postgresql.automigrate” which is useful when upgrading ChirpStack. Set it as you wish :

```sh
automigrate=true
```

#### AS ```redis``` section

If you changed the default port for redis or if you host redis on a different machine, don’t forget to change the ```redis.url``` parameter


#### AS ```application_server.external_api``` section

ChirpStack Application Server provides two API interfaces (```gRPC``` interface and RESTful ```JSON``` interface) that can be used to integrate with ChirpStack Application Server. Both interfaces provide exactly the same functionality and Authentication mechanism.

Both the gRPC and the JSON REST interface are protected by an authentication and authorization mechanism.  For this ```JSON web-tokens``` are used, using the ```--jwt-secret / JWT_SECRET``` value for signing. One can use a pre-defined token or [generate one JWT]

###	Starting the ChirpStack AS

To (re)start and stop ChirpStack As depends on if your distribution uses “init.d” or “systemd”:

init.d
```sh
sudo /etc/init.d/chirpstack-application-server [start|stop|restart|status]
```
systemd
```sh
sudo systemctl [start|stop|restart|status] chirpstack-application-server
```

###	Verifying the Functioning of the ChirpStack AS

init.d
```sh
tail -f /var/log/chirpstack-network-server/chirpstack-application-server.log
```
systemd
```sh
journalctl -u chirpstack-application-server -f -n 50
```
A successful start of the AS will have the following Output in the log file

```sh
Jul 15 21:33:43 vps323915 systemd[1]: Started ChirpStack Application Server.
Jul 15 21:33:43 vps323915 chirpstack-application-server[15346]: time="2020-07-15T21:33:43+02:00" level=info msg="starting ChirpStack Application Server" docs="https://www.chirpstack.io/" version=3.10.0
Jul 15 21:33:43 vps323915 chirpstack-application-server[15346]: time="2020-07-15T21:33:43+02:00" level=info msg="storage: setting up storage package"
Jul 15 21:33:43 vps323915 chirpstack-application-server[15346]: time="2020-07-15T21:33:43+02:00" level=info msg="storage: setup metrics"
Jul 15 21:33:43 vps323915 chirpstack-application-server[15346]: time="2020-07-15T21:33:43+02:00" level=info msg="storage: setting up Redis client"
Jul 15 21:33:43 vps323915 chirpstack-application-server[15346]: time="2020-07-15T21:33:43+02:00" level=info msg="storage: connecting to PostgreSQL database"
Jul 15 21:33:43 vps323915 chirpstack-application-server[15346]: time="2020-07-15T21:33:43+02:00" level=info msg="storage: applying PostgreSQL data migrations"
Jul 15 21:33:43 vps323915 chirpstack-application-server[15346]: time="2020-07-15T21:33:43+02:00" level=info msg="storage: PostgreSQL data migrations applied" count=0
Jul 15 21:33:43 vps323915 chirpstack-application-server[15346]: time="2020-07-15T21:33:43+02:00" level=info msg="integration/mqtt: TLS config is empty"
Jul 15 21:33:43 vps323915 chirpstack-application-server[15346]: time="2020-07-15T21:33:43+02:00" level=info msg="integration/mqtt: connecting to mqtt broker" server="tcp://localhost:1883"
Jul 15 21:33:43 vps323915 chirpstack-application-server[15346]: time="2020-07-15T21:33:43+02:00" level=info msg="api/as: starting application-server api" bind="0.0.0.0:8001" ca_cert= tls_cert= tls_key=
Jul 15 21:33:43 vps323915 chirpstack-application-server[15346]: time="2020-07-15T21:33:43+02:00" level=info msg="api/external: starting api server" bind="0.0.0.0:8080" tls-cert= tls-key=
Jul 15 21:33:43 vps323915 chirpstack-application-server[15346]: time="2020-07-15T21:33:43+02:00" level=info msg="integration/mqtt: connected to mqtt broker"
Jul 15 21:33:43 vps323915 chirpstack-application-server[15346]: time="2020-07-15T21:33:43+02:00" level=info msg="integration/mqtt: subscribing to tx topic" qos=0 topic=application/+/device/+/tx
Jul 15 21:33:43 vps323915 chirpstack-application-server[15346]: time="2020-07-15T21:33:43+02:00" level=info msg="api/external: registering rest api handler and documentation endpoint" path=/api
Jul 15 21:33:43 vps323915 chirpstack-application-server[15346]: time="2020-07-15T21:33:43+02:00" level=info msg="api/js: starting join-server api" bind="0.0.0.0:8003" ca_cert= tls_cert= tls_key=
...
...
```

*For now, if you followed all the above indications. You should have a running ChirpStack stack. Running the Network Server and Application Server with this configuration should give you access to the web-interface accessible using your AS’s IP default port (http://192.168.1.2:8080) or if you are running locally http://localhost:8080*


## Post Sanity Check from RGW->NS->AS Setup

The objective in this section is to verify the communication between the RGW->NS (2) and the NS->AS (3) shown in the figure:

<p align="center">
  <img width="760" height="300" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig14.png?raw=true">
</p>

### Verify whether the NS receives data from the RGW

Depending on your OS, one of the following commands will show you the logs:

```sh
journalctl -f -n 100 -u chirpstack-network-server
tail -f -n 100 /var/log/chirpstack-network-server/chirpstack-network-server.log
```
Expected log output

```sh
INFO[0163] backend/gateway: uplink frame received
INFO[0164] device gateway rx-info meta-data saved        dev_eui=0101010101010101
INFO[0164] device-session saved                          dev_addr=018f5aa9 dev_eui=0101010101010101
INFO[0164] finished client unary call                    grpc.code=OK grpc.method=HandleUplinkData grpc.service=as.ApplicationServerService grpc.time_ms=52.204 span.kind=client system=grpc
```

In this log, the NS has processed the uplink frame from the RGW, and forwarded the application payload to the AS

### Verify whether the AS receives data from the NS
Depending on your OS, one of the following commands will show you the logs:

```sh
journalctl -f -n 100 -u chirpstack-application-server
tail -f -n 100 /var/log/chirpstack-application-server/chirpstack-application-server.log
```
Expected log output

```sh
INFO[0186] handler/mqtt: publishing message              qos=0 topic=application/1/device/0101010101010101/rx
INFO[0186] finished unary call with code OK              grpc.code=OK grpc.method=HandleUplinkData grpc.request.deadline="2018-09-24T10:54:37+02:00" grpc.service=as.ApplicationServerService grpc.start_time="2018-09-24T10:54:36+02:00" grpc.time_ms=6.989 peer.address="[::1]:63536" span.kind=server system=grpc
```

In this log, the AS received an uplink application-payload from the NS and published this payload to the application/1/device/0101010101010101/rx MQTT topic.

[NS Setup]: #ns-setup
[AS Setup]: #as-setup
[Verifying Communication between RGW->NS->AS]: #post-sanity-check-from-rgw-ns-as-setup
[binary]:  https://www.chirpstack.io/network-server/overview/downloads/
[AS binary]: https://www.chirpstack.io/application-server/overview/downloads/
[generate one JWT]: https://www.chirpstack.io/application-server/integrate/auth/
[postgresql]: #postgresql-section
[redis]: #redis-section
[network_server]: #network_server-section
[metrics]: #metrics-section
[AS postgresql]
[AS redis]
[AS application_server.external_api]





