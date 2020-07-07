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

•	Multitech Conduit Gateway (Packet-forwarder + LoRa Gateway Bridge)
•	NS should have MQTT + Redis + PostgreSQL as pre-requisites
•	AS should have MQTT + Redis + PostgreSQL as pre-requisites


These are the ports you need to take into account for the firewall rules:

•	GW to server A (MQTT) (default 1883)
•	LoRa Server to LoRa App Server API (default 8001)
•	LoRa App Server to LoRa Server API (default 8000)
•	LoRa Server to LoRa App Server join-server API (default 8003)


Additionally, you probably want to expose the following ports too:

•	From your machine to AS web-interface (default port 8080)
•	From your machine to NS and AS MQTT (so that you can subscribe to the MQTT messages) (default port 1883)
