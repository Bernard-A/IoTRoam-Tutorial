```diff
-           In this document, We have used Multitech products as RGW. If you are using a different
-           Hardware you have to follow those Hardware specific details to set up the RGW
-           We are using the Chirpstack Gateway Bridge as the source: https://www.chirpstack.io/gateway-bridge/overview/
-           If you have a different source for the RGW set up, please follow the specific details
+           Please follow the Pointer section below for further details if you are using HW or SW different 
+           from mentioned above
```

# Multitech RGW Setup

This page covers setting up the RGW. The RGW recevies data from the ED over LoRa Radio Frequency. The RGW is responsible for transferring the received data to the NS over IP link in the required format. 

The setting up of the RGW is detailed in different sections as follows:

 * [Hardware Setup]
 * [Accessing the RGW]
 * [Configuring the Packet Forwarder]
 * [Install and Configuring the  Chirpstack Gateway bridge]
 * [Post Sanity check]


## Hardware Setup

For LoRa communication, the gateway needs a LoRa mCard (Figure 1) to be inserted into one of the two RF slots of the Multitech [MultiConnect Conduit] as illustrated in Figure 2

<p align="center">
  <img width="400" height="150" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig7.png?raw=true">
</p>

 *	Attach the 868 MHz antenna to the LoRa mCard’s RF port.
 *	Attach the cellular antenna to the Conduit’s CELL port. If there are two antennas, connect the second one in the place where it is written ‘Aux’
 *	Power the Conduit by plugging in the provided AC-DC converter
 *	Check the LEDs in the Conduit
    *	PWR = Power
    *	Status = LED blinks when the OS is fully loaded
    *	LS = Link Status
 * For detailed information follow the [Quick Start Guide] and a  [video tutorial]

 
## Accessing the RGW
 
There are multiple interfaces available to access the device - Serial ports, Ethernet and Cellular (if you buy the Multitech Conduit with Cellular antenna). Assuming that you are able to access (by watching the Mulitech links above or you have  solved your selves) to your RGW by an **SSH** and the **Web Interface**, the steps to follow are detailed in the following sections:

In addition, here is a brief pointer on the steps followed by us to access the Multitech RGW:

 * Connect to the USB device connection slot (found near the Antenna) in the Multitech Conduit RGW and then connect the other end to your computer
 * Similar to the [End Device Hardware Setup], identify the serial port interface after physically connecting to the serial port from one's computer
 * Once can get the IP address are explained [here], thus enabling to connect to the RGW via SSH or the Web interface
 * We are using the [Conduit AEP model] as firmware for the RGW. Hence it is possible to access the Web interface of the RGW connecting to the IP address from a web client.
 * If not done, do a [Firmware Upgrade] of the RGW  from the web interface
 * [Configure the AEP model]. Depending on the firmware, the front end of the configuration process might slightly change.


## Configuring the Packet Forwarder


If you relate to the [Architecture] page, our focus is on the two components as shown in the figure below: 

<p align="center">
  <img width="300" height="150" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig8.png?raw=true">
</p>

The function of **Packet Forwarder** is to forward the received LoRa Packet from the ED to the NS (which is identified by a fixed IP address and port). The function of the **GW bridge** is to pack the data sent by the packet forwarder into a specified format (e.g. JSON) and upload to the NS.

*The packet forwarder is a program running on the RGW that forwards RF packets receive by the concentrator to the NS through a IP/UDP link.*

```The source for the information below is :(https://www.chirpstack.io/gateway-bridge/gateway/multitech/#setting-up-the-packet-forwarder)```

The Mltitech RGW has a default Packet Forwarder. We have to enable that Packet forwarder option. 

 * We are using AEP version of the Multitech RGW. Hence we can enable this option once connected to the web interface. The options could change depending on the firmware version. For the RGW AEP Firmware version : 5.2.1, the steps are as follows: 
    *	 On the left menu, select LoRaWAN.
    *  Set “Mode” to “PACKET FORWARDER”.
    *	Other options are as shown in the figure below:

<p align="center">
  <img width="700" height="600" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig18.png?raw=true">
</p>


``` Note that the serv_port_up and serv_port_down represent the ports used to communicate with the chirpstack-gateway-bridge, usually on localhost (the server_address parameter). See the Figure above.```
 * Select  ```Submit ```.
 * Select the  ```Save and Apply``` option on the left menu.

## Install and Configuring the  Chirpstack Gateway(GW) bridge 
  
We are using use the Open Source ["Chirpstack"]. The first step is to install Chirpstack GW bridge software in the RGW.
 
*ChirpStack GW Bridge is a service which converts LoRa® Packet Forwarder protocols into a ChirpStack Network Server common data-format (JSON and Protobuf). This component is part of the ChirpStack open-source LoRaWAN® Network Server stack.*

```*The source for the information below is : (https://www.chirpstack.io/gateway-bridge/gateway/multitech/#setting-up-the-chirpstack-gateway-bridge)*```

 * Log in to the RGW using  ```SSH``` or use the  ```USB to serial interface ```
 * Download the latest chirpstack-gateway-bridge ```.ipk``` package
   ```sh
       wget https://artifacts.chirpstack.io/vendor/multitech/conduit/chirpstack-gateway-bridge_(latest_version_number).ipk
   ```
* Install the latest chirpstack-gateway-bridge 
     ```sh
          sudo opkg install chirpstack-gateway-bridge_(latest_version_number).ipk
     ```
* Update the MQTT<sup>[1](#myfootnote1)</sup> connection details so that ChirpStack Gateway Bridge is able to connect to your MQTT broker in your NS. The configuration file is ```/var/config/chirpstack-gateway-bridge/chirpstack-gateway-bridge.toml```. The modifications in the configuration file depends, but to minimum one has to provide the host IP and the port number of one's NS as follows: 
 ```sh
    [integration.mqtt.auth.generic]
    # MQTT server (e.g. scheme://host:port where scheme is tcp, ssl or ws)
    servers=[
      "tcp://127.0.0.1:1883",    ## To authorize the Mosquito client ublish to the MQTT topic   
                                 ## and the mosquitto_sub client is authorized to subscribe to the given MQTT topic
      "tcp://192.168.2.1:1883" ## Add your NS IP address 
    ]
  ```
* Start the chirpstack-gateway-bridge and you should get the confirmation that the gateway has started
    ```sh
        sudo /etc/init.d/chirpstack-gateway-bridge start
    ```
* Verify whether the chirpstack-gateway-bridge has started
    ```sh
        ps aux |grep chirpstack-gateway-bridge
    ```
    The Output should look something like this:
    ```sh
        root     32440  0.2  3.9 804684 10060 ?        Sl   17:50   0:00 /opt/chirpstack-gateway-bridge/chirpstack-gateway-bridge 
                                                        --config /var/config/chirpstack-gateway-bridge/chirpstack-gateway-bridge.toml

    ```
 
    
## Post Sanity check
```The source for the information below is :(https://www.chirpstack.io/guides/troubleshooting/gateway/)```

To verify the set up, we have to check the following:

* If the Packet Forwarder is receiving device data
* If the ChirpStack Gateway Bridge is receiving data from the packet-forwarder
* If ChirpStack Gateway Bridge is publishing the data to the MQTT broker

### Packet Forwarder is receiving and Sending data

When the ChirpStack Gateway Bridge is running on the gateway itself, then you need to run the following command on the gateway (it will monitor the loopback interface):
 ```sh
  sudo tcpdump -AUq -i lo port 1700
```
When the ChirpStack Gateway Bridge is installed on a separate machine / server, the you need to run the following command:
 ```sh
  sudo tcpdump -AUq port 1700
```
Expected Output 
```sh
  ...
  11:42:00.114726 IP localhost.34268 > localhost.1700: UDP, length 12
  E..(..@.@."................'.....UZ.....
  11:42:00.130292 IP localhost.1700 > localhost.34268: UDP, length 4
  E.. ..@.@.".....................
  11:42:10.204723 IP localhost.34268 > localhost.1700: UDP, length 12
  E..(.&@.@..................'.x...UZ.....
  ...
```
What we see in this log:
```sh
  localhost.130292 > localhost.1700: packet sent from the packet-forwarder to the ChirpStack Gateway Bridge
  localhost.1700 > localhost.130292: packet sent from the ChirpStack Gateway Bridge to the Packet Forwarder
```

### ChirpStack Gateway Bridge is receiving data from the packet-forwarder

A log file could be added for the GW bridge by [customising]. Otherwise one could stop and start the GW bridge manually by pointing to the Config file as follows:
```sh
   /opt/chirpstack-gateway-bridge/chirpstack-gateway-bridge -c /var/config/chirpstack-gateway-bridge/chirpstack-gateway-bridge.toml
```
**Expected Output**

```sh
INFO[0001] starting ChirpStack Gateway Bridge            docs="https://www.chirpstack.io/gateway-bridge/" version=3.4.0
INFO[0001] backend/semtechudp: starting gateway udp listener  addr="0.0.0.0:1700"
INFO[0001] integration/mqtt: connected to mqtt broker   
INFO[0008] integration/mqtt: subscribing to topic        qos=0 topic="gateway/00800000a0000825/command/#"
INFO[0075] integration/mqtt: publishing event            event=up qos=0 topic=gateway/00800000a0000825/event/up uplink_id=ef7a33fb-d83f-44d8-9e80-59c67427bbaa
```

When the Packet Forwarder sends data to the ChirpStack Gateway Bridge (this could be a “ping”), you will see the following logs:
```sh
  INFO[0008] integration/mqtt: subscribing to topic        qos=0 topic="gateway/00800000a0000825/command/#"
```
When your device sends an uplink message, you will see something like:

```sh
INFO[0075] integration/mqtt: publishing event            event=up qos=0 topic=gateway/00800000a0000825/event/up uplink_id=ef7a33fb-d83f-44d8-9e80-59c67427bbaa
```
If you see these logs, then this indicates that the ChirpStack Gateway Bridge components receives the data sent by the packet-forwarder.


### ChirpStack Gateway Bridge is publishing the data to the MQTT broker

<p align="center">
  <img width="300" height="150" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig17.png?raw=true">
</p>

To validate that the ChirpStack Gateway Bridge is publishing LoRa® frames to the MQTT broker, you can subscribe to the ```gateway/#``` MQTT [topic]. When using the mosquitto_sub utility, you can execute the following command:

```sh
mosquitto_sub -v -t "gateway/#"
```
As soon as the ED sends data, the output for the above command is as follows:
```sh
gateway/00800000a0000825/event/up {"phyPayload":"APliDNaluVYV260AAAAAgABv6D9SA8I=","txInfo":{"frequency":868100000,"modulation":"LORA","loRaModulationInfo":{"bandwidth":125,"spreadingFactor":12,"codeRate":"4/5","polarizationInversion":false}},"rxInfo":{"gatewayID":"AIAAAKAACCU=","time":"2020-05-27T04:28:10.184600Z","timeSinceGPSEpoch":null,"rssi":-102,"loRaSNR":7.2,"channel":0,"rfChain":0,"board":0,"antenna":0,"location":null,"fineTimestampType":"NONE","context":"XcwXjA==","uplinkID":"B1Oy7C2EQbuBKyLBTHrNuQ=="}}
```

When you do not see any data appear when your device sends data, then make sure the ChirpStack Gateway Bridge is authorized to publish to the MQTT topic and the mosquitto_sub client is authorized to subscribe to the given MQTT topic. This issue usually happens when you have configured your MQTT broker so that clients need to authenticate when connecting.


## Footnotes
<a name="myfootnote1">1</a>: Message Queuing Telemetry Transport protocol (MQTT) protocol, is a message transmission protocol based on the lightweight, publish-subscribe network model.  MQTT protocol is perfectly applied into the IoT solutions with very “Low-bandwidth and unreliable links”.  The RGW is MQTT enabled that support MQTT forwarding which runs over the TCP/IP and provides ordered, lossless, and bi-directional connections.


[Quick Start Guide]: https://www.multitech.com/documents/publications/quick-start-guides/82101452L-Conduit-Quick-Start.pdf 
[video tutorial]: https://www.multitech.net/developer/software/lora/getting-started-with-lora-conduit-aep/
["Chirpstack"]: https://www.chirpstack.io
[Architecture]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Architecture.md
[Hardware Setup]: #Hardware-Setup
[Accessing the RGW]: #Accessing-the-RGW
[Configuring the Packet Forwarder]: #Configuring-the-Packet-Forwarder
[Install and Configuring the  Chirpstack Gateway bridge]: #install-and-configuring-the--chirpstack-gatewaygw-bridge
[Post Sanity check]: #Post-Sanity-check
[MultiConnect Conduit]: https://www.multitech.com/brands/multiconnect-conduit
[End Device Hardware Setup]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/End-Device.md#hardware-setup
[Firmware Upgrade]: http://www.multitech.net/developer/downloads/#aep
[here]: https://www.chirpstack.io/gateway-bridge/gateway/multitech/
[Conduit AEP model]: https://www.multitech.net/developer/products/multiconnect-conduit-platform/conduit/
[Configure the AEP model]: https://www.thethingsnetwork.org/docs/gateways/multitech/aep.html
[latest]: https://artifacts.chirpstack.io/vendor/multitech/conduit/
[customising]: https://forum.chirpstack.io/t/no-logfile-from-lora-gateway-bridge/2925/7
[topic]: https://www.chirpstack.io/gateway-bridge/integrate/generic-mqtt/

