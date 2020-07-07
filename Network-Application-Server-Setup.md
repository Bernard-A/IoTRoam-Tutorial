
# Installing the Network Server and Application Server using ChirpStack in Debian/Ubuntu

This page covers setting up the RGW which are detailed in different sections as follows:

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
