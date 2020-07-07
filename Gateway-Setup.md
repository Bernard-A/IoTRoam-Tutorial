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
 
 There are multiple interfaces available to access the device - Serial ports, Ethernet and Cellular (if you buy the Multitech Conduit with Cellular antenna). Assuming that you are able to access (by watching the Mulitech links above or you have or solved your selves) to your RGW by an SSH connection, the steps to follow are detailed in the following subsections:
 
  ### Install and Configure the  Chirpstack GW bridge
  
  For this roaming tutorial we use the Open Source "Chirpstack"
 
ChirpStack Gateway Bridge is a service which converts LoRa® Packet Forwarder protocols into a ChirpStack Network Server common data-format (JSON and Protobuf). This component is part of the ChirpStack open-source LoRaWAN® Network Server stack.





[For detailed information follow the Quick Start Guide]: https://www.multitech.com/documents/publications/quick-start-guides/82101452L-Conduit-Quick-Start.pdf 
 [A video tutorial could be found here]: https://www.multitech.net/developer/software/lora/getting-started-with-lora-conduit-aep/
