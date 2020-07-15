
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
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1CE2AFD36DBCCA00
$ sudo echo "deb https://artifacts.chirpstack.io/packages/3.x/deb stable main" | sudo tee /etc/apt/sources.list.d/chirpstack.list
$ sudo apt-get update
$ sudo apt-get install chirpstack-network-server
```

### Configuration for the ChirpStack Network Server
The configuration file is located in the chirpstack-network-server folder in /etc. It is a “toml“ file.
```sh
$ /etc/chirpstack-network-server/chirpstack-network-server.toml
```

ChirpStack recommends checking to following parameters in the above .toml file when setting up a ChirpStack Network Server:
1.	postgresql.dsn
2.	postgresql.automigrate
3.	redis
4.	network_server.net_id
5.	network_server.band.name 
6.	metrics.timezone

#### 1. Update the “dsn” parameter with the parameters you provided when setting up your own PostgreSQL database:
```sh
[postgresql]
.
.
dsn="postgres://chirpstack_ns:dbnspassword@localhost/chirpstack_ns?sslmode=disable"
``` 

#### 2.	The following parameter in the default configuration file is “postgresql.automigrate” which is useful when upgrading ChirpStack. Set it as you wish :
```sh
[postgresql]
.
.
automigrate=true
```

#### 3.	In the “redis” section. If you changed the default port for redis or if you host redis on a different machine, don’t forget to change the ```redis.url``` parameter

#### 4.	In the “network_server” section, you may set your LoRaWAN NetID (if you have one).  The net_id parameter may be set to “000000” or “000001” for experimental network. But such a network won’t allow passive roaming:
```sh
[network_server]
.
.
net_id=123456
```

#### 5.	Also check out if your LoRaWAN transmission band is consistent with the regulation for the region you live in (As we are located in Europe and use the 868MHz band, we set it in the configuration (other possible values are indicated in the default configuration file)
```sh
[network_server]
.
.
   [network_server.band]
   .
   .
   name=”EU_863_870”
```

#### 6.	In the “metrics” section, you can use either “local” for the system local time zone or use settings such as ```“Europe/Paris”```

### 	Starting the ChirpStack Network Server
To (re)start and stop ChirpStack Network Server depends on if your distribution uses “init.d” or “systemd”:

init.d
```sh
sudo /etc/init.d/chirpstack-network-server [start|stop|restart|status]
```
systemd
```sh
sudo systemctl [start|stop|restart|status] chirpstack-network-server
```

### 	Verifying the Functioning of the ChirpStack Network Server

init.d
```sh
tail -f /var/log/chirpstack-network-server/chirpstack-network-server.log
```
systemd
```sh
journalctl -u chirpstack-network-server -f -n 50
```
A successful start of the Network Server will have the following Output in the log file

<p align="center">
  <img width="500" height="150" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig9.png?raw=true">
</p>


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

Setup your PostgreSQL database for your ChirpStack Network Server
```sh
$ sudo -u postgres psql
> create role chirpstack_as with login password 'dbaspassword';
> create database chirpstack_as with owner chirpstack_as;
> \c chirpstack_as
> create extension pg_trgm;
> create extension hstore;
> \q
```

Verify the set up
```sh
$ psql -h localhost -U chirpstack_as -W chirpstack_as
```

###	Obtaining and Installing the ChirpStack Application Server
The binary for ChirpStack Application Server is accessible following the link: https://www.chirpstack.io/application-server/overview/downloads/
Setup the Application Server, either using the binary from the link above, or by using Debian package manager. To install from the package manager, you will also need “apt-transport-https” to connect to the repository

```sh
$ sudo apt install apt-transport-https
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1CE2AFD36DBCCA00
$ sudo echo "deb https://artifacts.chirpstack.io/packages/3.x/deb stable main" | sudo tee /etc/apt/sources.list.d/chirpstack.list
$ sudo apt-get update
$ sudo apt-get install chirpstack-application-server
```

###	Configuration for the ChirpStack Application Server
The configuration file is located in the chirpstack-application-server folder in /etc. It is a “toml“ file.

```sh
$ /etc/chirpstack-network-server/chirpstack-application-server.toml
```
ChirpStack recommends checking to following parameters in the above .toml file when setting up a ChirpStack Application Server:
1.	postgresql.dsn  
2.	postgresql.automigrate
3.	redis

### 1.	Update the “dsn” parameter with the parameters you provided when setting up your own PostgreSQL database:

```sh
[postgresql]
.
.
dsn="postgres://chirpstack_as:dbnspassword@localhost/chirpstack_as?sslmode=disable"
```

### 2.	The following parameter in the default configuration file is “postgresql.automigrate” which is useful when upgrading ChirpStack. Set it as you wish :

```sh
[postgresql]
.
.
automigrate=true
```

### 3.	In the “redis” section. If you changed the default port for redis or if you host redis on a different machine, don’t forget to change the ```redis.url``` parameter


###	Starting the ChirpStack Application Server

To (re)start and stop ChirpStack Application Server depends on if your distribution uses “init.d” or “systemd”:

init.d
```sh
sudo /etc/init.d/chirpstack-application-server [start|stop|restart|status]
```
systemd
```sh
sudo systemctl [start|stop|restart|status] chirpstack-application-server
```

###	Verifying the Functioning of the ChirpStack Application Server

init.d
```sh
tail -f /var/log/chirpstack-network-server/chirpstack-application-server.log
```
systemd
```sh
journalctl -u chirpstack-application-server -f -n 50
```
A successful start of the Network Server will have the following Output in the log file

<p align="center">
  <img width="600" height="300" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig10.png?raw=true">
</p>

*For now, if you followed all the above indications. You should have a running ChirpStack stack. Running the Network Server and Application Server with this configuration should give you access to the web-interface accessible using your Application Server’s IP default port (http://A.B.C.D:8080) or if you are running locally http://localhost:8080*


## Verify Communication from RGW->NS->AS Setup

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
[Verifying Communication between RGW->NS->AS]: #verify-communication-from-rgw-ns-as-setup
[binary]:  https://www.chirpstack.io/network-server/overview/downloads/
