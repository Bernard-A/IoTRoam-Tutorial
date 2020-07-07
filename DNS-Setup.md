This page covers the following: 
Why DNS? 
How to Provision in the DNS for OTAA and Roaming?

# Why the DNS infrastructure is required in the LoRaWAN set up?

As per the LoRaWAN Backend specifications, the DNS infrastructure is used for the following operations:

1.	OTAA: By the NS to get the IP address of the Join Server
2.  Roaming: By the vNS to find the hNS
3.  OTAA: Associate a batch of devices to one specific Join Server using a DNS look-up, combining DevEUI and JoinEUI

Two types of identifiers are provisioned in the DNS - 1. NetIDS for Roaming and 2. JoinEUI for OTAA. as per the backend specification, the LoRa Alliance DNS tree is as shown in the below figure:

<p align="center">
  <img width="760" height="400" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig11.png?raw=true">
</p>

As of now, the LoRa DNS service is not operational. Hence, for the Academic LoRaWAN roaming purposes we have a separate zone called ```iotreg.net``` where the NetIDs and JoinEUIs could be provisioned as shown in the figure below:

<p align="center">
  <img width="760" height="400" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig12.png?raw=true">
</p>

