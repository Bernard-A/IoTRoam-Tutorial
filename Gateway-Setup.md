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
 
There are multiple interfaces available to access the device - Serial ports, Ethernet and Cellular (if you buy the Multitech Conduit with Cellular antenna). Assuming that you are able to access (by watching the Mulitech links above or you have or solved your selves) to your RGW by an SSH connection, the steps to follow are detailed in the following sections:

If you relate to the [Architecture] page, our focus is on the two components as shown in the figure below: 

<p align="center">
  <img width="300" height="150" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig8.png?raw=true">
</p>

The function of **Packet Forwarder** is to forward the received LoRa Packet from the ED to the NS (which is identified by a ixed IP address and port). The function of the **GW bridge** is to pack the data sent by the packet forwarder into a specified format (e.g. JSON) nd upload to the NS.

## Install and Configuring the  Chirpstack Gateway(GW) bridge
  
For this roaming tutorial we use the Open Source ["Chirpstack"]. The first step is to install Chirpstack GW bridge software in the RGW.
 
*ChirpStack GW Bridge is a service which converts LoRa® Packet Forwarder protocols into a ChirpStack Network Server common data-format (JSON and Protobuf). This component is part of the ChirpStack open-source LoRaWAN® Network Server stack.*

```*The source for the information below is : (https://www.chirpstack.io/gateway-bridge/gateway/multitech/#setting-up-the-chirpstack-gateway-bridge)*```

 * In the RGW, install Chirpstack GW bridge
    * Log in o the RGW using SSH or use the USB to serial interface.
    * Download the latest chirpstack-gateway-bridge .ipk package from: https://artifacts.chirpstack.io/vendor/multitech/conduit/. Example (assuming you want to install chirpstack-gateway-bridge_3.1.0-r1_arm926ejste.ipk): [```Download the package based upon your RGW vendor```]
     ```sh
          wget https://artifacts.chirpstack.io/vendor/multitech/conduit/chirpstack-gateway-bridge_3.1.0-r1_arm926ejste.ipk 
     ```
    * Now that the .ipk package is stored on the Conduit, you can install it using the opkg package-manager utility. Example (assuming the same .ipk file):
     ```sh
          opkg install chirpstack-gateway-bridge_3.1.0-r1_arm926ejste.ipk
     ```
    * Update the MQTT connection details so that ChirpStack Gateway Bridge is able to connect to your MQTT broker. You will find the configuration file in the ```/var/config/chirpstack-gateway-bridge/chirpstack-gateway-bridge.toml``` directory.
    * Start ChirpStack Gateway Bridge and ensure it will be started on boot. Example:
     ```sh
      /etc/init.d/chirpstack-gateway-bridge start
      update-rc.d chirpstack-gateway-bridge defaults
     ```

## Install and Configuring the Chirpstack Packet Forwarder

*The packet forwarder is a program running on the RGW that forwards RF packets receive by the concentrator to the NS through a IP/UDP link.*

The Mltitech RGW has a default Packet Forwarder. We have to disable that Packet forwarder and set up the Chirpstack Packet Forwarder. 
```*The source for the information below is :(https://www.chirpstack.io/gateway-bridge/gateway/multitech/#setting-up-the-packet-forwarder)*```





[For detailed information follow the Quick Start Guide]: https://www.multitech.com/documents/publications/quick-start-guides/82101452L-Conduit-Quick-Start.pdf 
[A video tutorial could be found here]: https://www.multitech.net/developer/software/lora/getting-started-with-lora-conduit-aep/
["Chirpstack"]: https://www.chirpstack.io
[Architecture]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Architecture.md
