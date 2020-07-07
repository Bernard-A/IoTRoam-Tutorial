# Architecture

Figure 1 shows the basic building blocks of a LoRaWAN Roaming infrastructure. The different building blocks involved are:
1.	End-Device (ED)
2.	Radio Gateway (RGW)
3.	Network Server (NS)
4.	Application Server (AS)
5.	Domain Naming System (DNS) Database 

![alt text](https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig1.png?raw=true)

In this document, We have used Multitech products for ED and the RGW. If you are using a
different Hardware you have to follow those Hardware specific details to set up the 
ED and RGW. The scenario as per Fig 1 is as follows:
 *	Multitech Conduit Gateway (Packet-forwarder + LoRa Gateway Bridge)
 *	NS should have  as pre-requisites the following installed : MQTT, Redis and PostgreSQL
 *	AS should have  as pre-requisites the following installed : MQTT, Redis and PostgreSQL

These are the ports you need to take into account for the firewall rules:
 *	GW => NS  (default is Port 1883 for MQTT))
 *	NS => AS (default is Port 8001)
 *	AS => NS (default is Port 8000)
 *	NS => AS to (default is Port 8003) #This is required if your AS and Join Server (JS) 
                                        are same machines

Additionally, you probably want to expose the following ports too:
 *	From your computer to AS web-interface (default is port 8080)
 *	From your computer to NS and AS MQTT (so that you can subscribe to the MQTT messages) (default is port 1883)
