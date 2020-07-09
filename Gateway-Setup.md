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
 * [For detailed information follow the Quick Start Guide] 
 * [A video tutorial could be found here] 
 
## Accessing the RGW
 
There are multiple interfaces available to access the device - Serial ports, Ethernet and Cellular (if you buy the Multitech Conduit with Cellular antenna). Assuming that you are able to access (by watching the Mulitech links above or you have  solved your selves) to your RGW by an **SSH** and the **Web Interface**, the steps to follow are detailed in the following sections:

In addition, here is a brief pointer on the steps followed by us to access the Multitech RGW:

 * Connect to the USB device connection slot (found near the Antenna) in the Multitech Conduit RGW and then connect to your computer
 * Similar to the [End Device Hardware Setup], identify and connect to the serial port from one's computer
 * Get the IP address are explained [here]
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

 * We are using AEP version of the Multitech RGW. Hence we can enable this option once connected to the web interface 
    *	 On the left menu, select “Setup”, and then “LoRa Network Server” from the submenu.
    *	In the “LoRa Configuration” window:
    *	At the top of the left column, check “Enabled”.
    *	At the top of the right column, set “Mode” to be “PACKET FORWARDER”.
    *	In the “Config” text box, modify/add the following configuration details in the gateway_conf section.):
   ```sh
    {
        ...
        "gateway_conf": {
            ...
            "server_address": "localhost",
            "serv_port_up": 1700,
            "serv_port_down": 1700,
             ... 
        }
    }
   ```    
``` Note that the serv_port_up and serv_port_down represent the ports used to communicate with the chirpstack-gateway-bridge, usually on localhost (the server_address parameter). See the Figure above.```
 * Select  ```Submit ```.
 * Select the  ```Save and Restart ``` option on the left menu.

## Install and Configuring the  Chirpstack Gateway(GW) bridge 
  
We are using use the Open Source ["Chirpstack"]. The first step is to install Chirpstack GW bridge software in the RGW.
 
*ChirpStack GW Bridge is a service which converts LoRa® Packet Forwarder protocols into a ChirpStack Network Server common data-format (JSON and Protobuf). This component is part of the ChirpStack open-source LoRaWAN® Network Server stack.*

```*The source for the information below is : (https://www.chirpstack.io/gateway-bridge/gateway/multitech/#setting-up-the-chirpstack-gateway-bridge)*```

 * Log in o the RGW using  ```SSH ``` or use the  ```USB to serial interface ```
 * Download the latest chirpstack-gateway-bridge ```.ipk``` package
   ```sh
       wget https://artifacts.chirpstack.io/vendor/multitech/conduit/chirpstack-gateway-bridge_(latest_version_number).ipk
   ```
* Install the latest chirpstack-gateway-bridge 
     ```sh
          sudo opkg install chirpstack-gateway-bridge_(latest_version_number).ipk
     ```
* Update the MQTT connection details so that ChirpStack Gateway Bridge is able to connect to your MQTT broker. You will find the configuration file in the ```/var/config/chirpstack-gateway-bridge/chirpstack-gateway-bridge.toml``` directory.
* Start the latest chirpstack-gateway-bridge depending on your system 
    ```sh
        sudo /etc/init.d/chirpstack-gateway-bridge start
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

All logs are written to /var/log/chirpstack-gateway-bridge/chirpstack-gateway-bridge.log. To verify that the GW bridge is receiving data from the packet forwarder, view and follow this logfile depending on your system:
* init.d
```sh
   tail -f /var/log/chirpstack-gateway-bridge/chirpstack-gateway-bridge.log
```
* systemd
 ```sh
   journalctl -u chirpstack-gateway-bridge -f -n 50
```

Expected Output

When the Packet Forwarder sends data to the ChirpStack Gateway Bridge (this could be a “ping”), you will see the following logs:
```sh
  INFO[0013] mqtt: subscribing to topic qos=0 topic=gateway/7276ff002e062c18/command/#
```
When your device sends an uplink message, you will see something like:

```sh
  INFO[0267] mqtt: publishing message qos=0 topic=gateway/7276ff002e062c18/event/up
```
If you see these logs, then this indicates that the ChirpStack Gateway Bridge components receives the data sent by the packet-forwarder.


### ChirpStack Gateway Bridge is publishing the data to the MQTT broker

To validate that the ChirpStack Gateway Bridge is publishing LoRa® frames to the MQTT broker, you can subscribe to the gateway/# MQTT topic. When using the mosquitto_sub utility, you can execute the following command:

```sh
mosquitto_sub -v -t "gateway/#"
```

When you do not see any data appear when your device sends data, then make sure the ChirpStack Gateway Bridge is authorized to publish to the MQTT topic and the mosquitto_sub client is authorized to subscribe to the given MQTT topic. This issue usually happens when you have configured your MQTT broker so that clients need to authenticate when connecting.



[For detailed information follow the Quick Start Guide]: https://www.multitech.com/documents/publications/quick-start-guides/82101452L-Conduit-Quick-Start.pdf 
[A video tutorial could be found here]: https://www.multitech.net/developer/software/lora/getting-started-with-lora-conduit-aep/
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


