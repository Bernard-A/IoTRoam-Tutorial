# Multitech RGW Setup

This page covers the following operating systems:

 * [Hardware Setup]
 * [Accessing the RGW]
 * [Install and Configuring the Packet Forwarder]
 * [Install and Configuring the  Chirpstack Gateway(GW) bridge]
 * [Post Sanity check]


## Hardware Setup

For LoRa communication, the gateway needs a LoRa mCard (Figure 1) to be inserted into one of the two RF slots of the Multitech MultiConnect Conduit as illustrated in Figure 2

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

If you relate to the [Architecture] page, our focus is on the two components as shown in the figure below: 

<p align="center">
  <img width="300" height="150" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig8.png?raw=true">
</p>

The function of **Packet Forwarder** is to forward the received LoRa Packet from the ED to the NS (which is identified by a fixed IP address and port). The function of the **GW bridge** is to pack the data sent by the packet forwarder into a specified format (e.g. JSON) and upload to the NS.

## Install and Configuring the Packet Forwarder

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
 * Select “Submit”.
 * Select the “Save and Restart” option on the left menu.

## Install and Configuring the  Chirpstack Gateway(GW) bridge (Debian/Ubuntu)
  
For this roaming tutorial we use the Open Source ["Chirpstack"]. The first step is to install Chirpstack GW bridge software in the RGW.
 
*ChirpStack GW Bridge is a service which converts LoRa® Packet Forwarder protocols into a ChirpStack Network Server common data-format (JSON and Protobuf). This component is part of the ChirpStack open-source LoRaWAN® Network Server stack.*

```*The source for the information below is : (https://www.chirpstack.io/gateway-bridge/gateway/multitech/#setting-up-the-chirpstack-gateway-bridge)*```

 * Log in o the RGW using SSH or use the USB to serial interface
 * Activate the repository:
     ```sh
       sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1CE2AFD36DBCCA00 
       sudo echo "deb https://artifacts.chirpstack.io/packages/3.x/deb stable main" | sudo tee  /etc/apt/sources.list.d/chirpstack.list
       sudo apt-get update
     ```
* Install the latest chirpstack-gateway-bridge 
     ```sh
          sudo apt-get install chirpstack-gateway-bridge 
     ```
* Update the MQTT connection details so that ChirpStack Gateway Bridge is able to connect to your MQTT broker. You will find the configuration file in the ```/var/config/chirpstack-gateway-bridge/chirpstack-gateway-bridge.toml``` directory.
* Start the latest chirpstack-gateway-bridge depending on your system uses ```init.d``` or ```systemd```
    * init.d    
    ```sh
        sudo /etc/init.d/chirpstack-gateway-bridge [start|stop|restart|status]
    ```
    * systemd  
    ```sh
        systemctl [start|stop|restart|status] chirpstack-gateway-bridge
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


