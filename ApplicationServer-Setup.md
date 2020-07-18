<link rel="stylesheet" type="text/css" href="css/style.css">



# Installing and Configuring the Application Server (AS) using ChirpStack (Debian/Ubuntu)

<div class="h1">
  This text is purple, <a href="#" class="text-inherit">including the link</a>
</div>

In our set up, we have a dedicated AS Virtual Private Server which is accessible externally by distinct physical address. The JS functions in the same Virtual Private Server as the AS in out set up. 

This page covers setting up the AS which are detailed in different sections as follows:

 * [AS Setup]
 * [Web Interface Setup]
 * [Verifying Communication between RGW->NS->AS]


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

* 1. [postgresql]
* 2. [redis]
* 3. [application_server.external_api]

#### ```postgresql``` section

Update the “dsn” parameter with the parameters you provided when setting up your own PostgreSQL database:

```sh
dsn="postgres://chirpstack_as:dbaspassword@localhost/chirpstack_as?sslmode=disable"
```

The following parameter in the default configuration file is “postgresql.automigrate” which is useful when upgrading ChirpStack. Set it as you wish :

```sh
automigrate=true
```

####  ```redis``` section

If you changed the default port for redis or if you host redis on a different machine, don’t forget to change the ```redis.url``` parameter


#### ```application_server.external_api``` section

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

If you see the last line from the ouput after starting the NS, it is as follows:

```sh
Jul 15 21:33:43 vps323915 chirpstack-application-server[15346]: time="2020-07-15T21:33:43+02:00" level=info msg="api/js: starting join-server api" bind="0.0.0.0:8003" ca_cert= tls_cert= tls_key=
```
It means that the JS is bound to port ```8003``` on all interfaces.

If you have followed the sequence of this tutorial, if you send data from the LoRa ED, you will not see any data in the AS logs. Hence, certain actions should be done in the AS web interface.

```sh 
If you followed the sequence of this tutorial, you should have a running ChirpStack stack. Starting the NS and AS 
should give you access to the web-interface using your AS’s IP and the default port as :http://192.168.1.2:8080
(or) if you are running locally as :http://localhost:8080
```

## Web Interface Setup

[Chirpstack UI page] provides a detailed description on how to set the web interface? Here we will just point out to those links in a sequential manner: 

* [Login] to your Web interface
* Set up or update [User], if required
* Add your NS in the Web interface which is self explanatory. At this juncture, fill only the ```General``` section
* Add/Update [Organisation]
* Click on the [Service Profile] and Update (click on ```Add gateway meta-data``` ; set ```Minimum allowed data-rate``` to ```0``` and ```Maximum allowed data-rate``` to ```5```)
* Click on the [Device Profile] and Update (Regional Paramater set to ```A``` and for others it is self explanatory)
   * In the tab ```Join OTAA/ABP```, click on Device Supports OTAA
   * I have not selected other tabs
* Create [Gateway]. Other than retrieving the Gateway ID, everything is self-explanatory
* One can verify that the GW is configured correctly by clicking on ```LIVE LORAWAN FRAMES``` in the Gateways menu and one should see the Join Request if you start sending data from your ED as follows:

<p align="center">
  <img width="760" height="200" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig19.png?raw=true">
</p>

* Click on the ```Applications``` in the menu and fill the required fields and Update
* Once again click on the ```Applications``` and click on the your Application Name
   * One has to set the DevEUI (which is obtained from your ED). In the [ED Setup], it is explained how to access your mDoT ED and type the following command to obtain the DevEUI
  ```sh
                  AT+DI       ## Unique Device EUI set at factory (8 bytes)
  ```
 * Once one click on the device that has been created you will come to a page and now as one clicks on the menu ```Keys(OTAA)``` the page will be displayed as below
 
 <p align="center">
  <img width="760" height="300" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig20.png?raw=true">
</p>

 * We have set only the  ```Application Key``` field. We clicked the arrow symbol in the right to generate an application key. You can also set your own Application Key as per the specifications. This key should be introduced to your ED. In  the mDot, we added using the following command:
  ```sh
                  AT+NK=0,DACFF1F919CC7BD3E7490439D926A3BD       ## Your App Key 
  ```
  * If you click on the ```Activation``` menu, at this instance you should get a response that the device is not activated as follows:
  
 <p align="center">
  <img width="760" height="125" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig21.png?raw=true">
</p>


## Post Sanity Check from RGW->NS->AS Setup

The objective in this section is to verify the communication between the RGW->NS (2) and the NS->AS (3) shown in the figure:

<p align="center">
  <img width="760" height="300" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig14.png?raw=true">
</p>

### Verify whether the NS receives data from the RGW

When the ED sends an uplink, the logs from the RGW is same as in the [GW-Setup]. There are two tools in the NS to check whether we have received any data from the RGW. 
 * First is the MQTT client in the NS and the logs after running the command ```mosquitto_sub -v -t "#"``` are as follows: 

```sh
gateway/00800000a0000825/event/up {"phyPayload":"APliDNaluVYV260AAAAAgAAnnH6k4Y0=","txInfo":{"frequency":868300000,"modulation":"LORA","loRaModulationInfo":{"bandwidth":125,"spreadingFactor":12,"codeRate":"4/5","polarizationInversion":false}},"rxInfo":{"gatewayID":"AIAAAKAACCU=","time":"2020-05-26T14:36:17.347455Z","timeSinceGPSEpoch":null,"rssi":-108,"loRaSNR":-5.8,"channel":1,"rfChain":0,"board":0,"antenna":0,"location":null,"fineTimestampType":"NONE","context":"OcO85A==","uplinkID":"PwbzZ213Rou7EoX+DkG5MQ=="}}
```
The above log confirms that there is a uplink payload recieved from the Gateway ```00800000a0000825```

 * Second is the log files from your NS. Launch a terminal with the below command as per your OS distribution
```sh
journalctl -f -n 100 -u chirpstack-application-server
tail -f -n 100 /var/log/chirpstack-application-server/chirpstack-application-server.log
```

Shows the logs as follows:
```sh
Jul 18 00:28:53 vps323914 chirpstack-network-server[21156]: time="2020-07-18T00:28:53+02:00" level=info msg="gateway/mqtt: uplink frame received" gateway_id=00800000a0000825 uplink_id=3f06f367-6d77-468b-bb12-85fe0e41b931
Jul 18 00:28:53 vps323914 chirpstack-network-server[21156]: time="2020-07-18T00:28:53+02:00" level=info msg="uplink: frame(s) collected" ctx_id=bee0644a-cf32-4179-8361-b47106df4898 mtype=JoinRequest uplink_ids="[3f06f367-6d77-468b-bb12-85fe0e41b931]"
Jul 18 00:28:53 vps323914 chirpstack-network-server[21156]: time="2020-07-18T00:28:53+02:00" level=warning msg="creating insecure application-server client" server="localhost:8001"
Jul 18 00:28:53 vps323914 chirpstack-network-server[21156]: time="2020-07-18T00:28:53+02:00" level=warning msg="grpc: addrConn.createTransport failed to connect to {127.0.0.1:8001  <nil> 0 <nil>}. Err: connection error: desc = \"transport: Error while dialing dial tcp 127.0.0.1:8001: connect: connection refused\". Reconnecting..."
Jul 18 00:28:54 vps323914 chirpstack-network-server[21156]: time="2020-07-18T00:28:54+02:00" level=error msg="uplink/join: get as client for routing-profile id error" ctx_id=bee0644a-cf32-4179-8361-b47106df4898 dev_eui=008000000000addb error="get application-server client error: create application-server api client error: dial application-server api error: context deadline exceeded"
Jul 18 00:28:54 vps323914 chirpstack-network-server[21156]: time="2020-07-18T00:28:54+02:00" level=error msg="uplink: processing uplink frame error" ctx_id=bee0644a-cf32-4179-8361-b47106df4898 error="join-request to join-server error: http post error: Post http://localhost:8003: dial tcp 127.0.0.1:8003: connect: connection refused"
```

If one looks carefully in the logs above, there is a ```warning``` and ```three error``` messages:
* Warning message is that the NS is ```creating insecure application-server client```. The reason being until now we have not set the TLS certificates. We will do it later
* First Error message is ```tcp 127.0.0.1:8001: connect: connection refused```
* Second Error message is `that since the connection is refused for port 8001, the NS is not able to get the client routing profile 
* Third error message is not able to connect to the Join Server using port 8003

To rectify the above issues:
* The connection refused for ports  ```8001``` and ```8003```. This where we redirect you to the [Architecture page] to make sure that required ports are open
* Make sure that the NS [JS-config] is pointing to the appropriate JS IP and port  

Once the above corrections are done, and on restarting both the NS and the AS, the logs are as follows:

```sh
Jul 18 12:28:08 vps323914 chirpstack-network-server[3524]: time="2020-07-18T12:28:08+02:00" level=info msg="gateway/mqtt: uplink frame received" gateway_id=00800000a0000825 uplink_id=f64a3e1f-a63d-4f5d-ba85-c235144b80c2
Jul 18 12:28:08 vps323914 chirpstack-network-server[3524]: time="2020-07-18T12:28:08+02:00" level=info msg="uplink: frame(s) collected" ctx_id=034b33b3-0389-4325-8629-a6277b6834f5 mtype=JoinRequest uplink_ids="[f64a3e1f-a63d-4f5d-ba85-c235144b80c2]"
Jul 18 12:28:08 vps323914 chirpstack-network-server[3524]: time="2020-07-18T12:28:08+02:00" level=warning msg="creating insecure application-server client" server="149.202.57.54:8001"
Jul 18 12:28:08 vps323914 chirpstack-network-server[3524]: time="2020-07-18T12:28:08+02:00" level=info msg="finished client unary call" ctx_id=034b33b3-0389-4325-8629-a6277b6834f5 grpc.code=OK grpc.ctx_id=93cfd556-1d3c-43fb-8811-0234fcf5cf81 grpc.duration=21.5717ms grpc.method=HandleError grpc.service=as.ApplicationServerService span.kind=client system=grpc
Jul 18 12:28:08 vps323914 chirpstack-network-server[3524]: time="2020-07-18T12:28:08+02:00" level=error msg="uplink: processing uplink frame error" ctx_id=034b33b3-0389-4325-8629-a6277b6834f5 error="join-request to join-server error: response error, code: MICFailed, description: invalid mic"
```

The error message that rests in the above log is ```invalid mic```



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

## Pointer Section

In the event of one using As that is not Chirpstack, please make sure that you [] . Once verified, Next section to follow : []

If you are using Chirpstack NS, Next section to follow : []

 * In any case, If you want to go back to the [Readme Page]
 * In any case, If you want to go back to the [Architecture page]
 * In any case, If you want to go back to the [Setting up the End-Device]
 * In any case, If you want to go back to the [Setting up the GW]
 * In any case, If you want to go back to the [NS Setup]


[AS Setup]: #as-setup
[Verifying Communication between RGW->NS->AS]: #post-sanity-check-from-rgw-ns-as-setup
[binary]:  https://www.chirpstack.io/network-server/overview/downloads/
[AS binary]: https://www.chirpstack.io/application-server/overview/downloads/
[generate one JWT]: https://www.chirpstack.io/application-server/integrate/auth/
[postgresql]: #postgresql-section
[redis]: #redis-section
[application_server.external_api]: #application_serverexternal_api-section
[Web Interface Setup]: #web-interface-setup 
[Chirpstack UI page]: https://www.chirpstack.io/application-server/use/
[Login]: https://www.chirpstack.io/application-server/use/login/
[User]: https://www.chirpstack.io/application-server/use/users/
[Organisation]: https://www.chirpstack.io/application-server/use/organizations/
[Service Profile]: https://www.chirpstack.io/application-server/use/service-profiles/
[Device Profile]: https://www.chirpstack.io/application-server/use/device-profiles/
[Gateway]: https://www.chirpstack.io/application-server/use/gateways/
[ED Setup]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/End-Device.md#hardware-setup
[Readme Page]: https://github.com/sandoche2k/IoTRoam-Tutorial
[Architecture page]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Architecture.md
[Setting up the End-Device]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/End-Device.md
[Setting up the GW]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Gateway-Setup.md
[NS Setup]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/NetworkServer-Server-Setup.md
[GW-Setup]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Gateway-Setup.md#chirpstack-gateway-bridge-is-receiving-data-from-the-packet-forwarder
[JS-config]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/NetworkServer-Server-Setup.md#Join-Server-section







