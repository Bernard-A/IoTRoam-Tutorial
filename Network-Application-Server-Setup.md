
# Installing and Configuring the Network Server and Application Server using ChirpStack (Debian/Ubuntu)

This page covers setting up the Network and Application Server which are detailed in different sections as follows:

 * [Network Server Setup]
 * [Application Server Setup]

## Network Server Setup

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

Setup your PostgreSQL database for your ChirpStack Network Server

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

###	Obtaining and Installing the ChirpStack Network Server

The binary for ChirpStack Network Server is accessible following the link: https://www.chirpstack.io/network-server/overview/downloads/

Setup the Network Server, either using the binary from the link above, or by using Debian package manager. To install from the package manager, you will also need “apt-transport-https” to connect to the repository.

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
A successful start of the Network Server will have the following O/P in the log file

<p align="center">
  <img width="600" height="150" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig9.png?raw=true">
</p>


