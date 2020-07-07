
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

Verify the set up

```sh
$ psql -h localhost -U chirpstack_ns -W chirpstack_ns
```
