# Architecture

Figure 1 shows the basic building blocks for setting up a Roaming LoRaWAN plateform. The different building blocks involved are:
1.	End-Device (ED)
2.	Radio Gateway (RGW)
3.	Network Server (NS)
4.	Application Server (AS)
5.	Domain Naming System (DNS) Database 

<p align="center">
  <img width="860" height="500" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig1.png?raw=true">
</p>

The scenario as per Fig 1 is as follows:
 *  The ED is capable of sending data over LoRa radio network
 *	RGW which needs to be set up with a *Packet-forwarder* and *GW Bridge*
 *	NS should have  as pre-requisites the following installed : MQTT, Redis and PostgreSQL
 *	AS should have  as pre-requisites the following installed : MQTT, Redis and PostgreSQL
 *  NetIDs and JoinEUIs provisioned in the DNS and it is publicly accessible over the Internet

To work effectively, one should also take into account for the firewall rules so that the ports are open between the different interfaces :
 *	GW => NS  (default is Port 1883 for MQTT)
 *	AS => NS (default is Port 8000)
 *  AS => NS (default is Port 1883 for MQTT) ``` As we used the NS as common MQTT brocker ```
 *	NS => AS (default is Port 8001)
 *	NS => AS to (default is Port 8003)  ``` This is required if your AS and Join Server (JS) are same machines ```

Additionally, you probably want to expose the following ports too:
 *	From your computer to AS web-interface (default is port 8080)
 *	From your computer to NS and AS (so that you can subscribe to the MQTT messages) (default is port 1883)
 
 
## Pointer Section 
Next section to follow: [Setting up the End-Device]

If you want to go back to the [Readme Page]

[Setting up the End-Device]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/End-Device.md
[Readme Page]: https://github.com/sandoche2k/IoTRoam-Tutorial 
