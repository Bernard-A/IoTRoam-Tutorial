# Multitech RGW Setup

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

To verify the set up, we have to check the following:

* If the Packet Forwarder is receiving device data
* If the ChirpStack Gateway Bridge is receiving data from the packet-forwarder
* If ChirpStack Gateway Bridge is publishing the data to the MQTT broker


All logs are written to /var/log/chirpstack-gateway-bridge/chirpstack-gateway-bridge.log. To verify that the GW bridge is receiving data from the packet forwarder, view and follow this logfile depending on your system:
* init.d
```sh
   tail -f /var/log/chirpstack-gateway-bridge/chirpstack-gateway-bridge.log
```
* systemd
 ```sh
   journalctl -u chirpstack-gateway-bridge -f -n 50
```


[For detailed information follow the Quick Start Guide]: https://www.multitech.com/documents/publications/quick-start-guides/82101452L-Conduit-Quick-Start.pdf 
[A video tutorial could be found here]: https://www.multitech.net/developer/software/lora/getting-started-with-lora-conduit-aep/
["Chirpstack"]: https://www.chirpstack.io
[Architecture]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Architecture.md
