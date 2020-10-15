# LoRaWAN Roaming Platform Set up Tutorial
This document describes the architecture, prerequisites, installation, post-install sanity checking for setting up an **"Open LoRaWAN Roaming Platform"**. The idea is to mimic the [EduRoam] infrastructure for WiFi.

## Objective 
Objective of this tutorial is to help users to set up a LoRaWAN as per the [LoRaWAN Backend Specifications]. Ths End-Devices which is part of this network should be able to access the intended services when they roam in the coverage area of other LoRaWANs which has been set up with an open roaming set up as described in this tutorial. 

The requirement to be part of this open LoRaWAN roaming set up is to [provision] your devices information in the DNS and have the [certificates] generated from the Root CA.

## Structure

This document is structured and one can follow sequentially starting with ``` Architecture ```:

 * [Architecture]
 * [Setting up the End-Device]
 * [Setting up the Packet Forwarder and the Radio Gateway Bridge]
 * [Installing and Configuring the Network Server]
 * [Installing and Configuring the Application Server]
 * [Setting up the DNS infrastructure]
 * [OTAA via DNS]
 * [Passive roaming via DNS]
 
 ``` At the end of each page, there will be a pointer section to guide you further ```
 
## Pointer Section 
 
Next section to follow: [Architecture]



[Architecture]: https://github.com/afnic/IoTRoam-Tutorial/blob/master/Architecture.md
[Setting up the End-Device]: https://github.com/afnic/IoTRoam-Tutorial/blob/master/End-Device.md
[Setting up the Packet Forwarder and the Radio Gateway Bridge]: https://github.com/afnic/IoTRoam-Tutorial/blob/master/Gateway-Setup.md
[Installing and Configuring the Network Server]: https://github.com/afnic/IoTRoam-Tutorial/blob/master/NetworkServer-Server-Setup.md
[Installing and Configuring the Application Server]: https://github.com/afnic/IoTRoam-Tutorial/blob/master/ApplicationServer-Setup.md
[Setting up the DNS infrastructure]: https://github.com/afnic/IoTRoam-Tutorial/blob/master/DNS-Setup.md
[OTAA via DNS]: https://github.com/afnic/IoTRoam-Tutorial/blob/master/OTAA-Using-DNS.md
[provision]: https://github.com/afnic/IoTRoam-Tutorial/blob/master/DNS-Setup.md#how-to-provision-netids-and-joineuis-in-the-dns-for-otaa-and-roaming
[certificates]: https://github.com/afnic/IoTRoam-Tutorial/blob/master/OTAA.md#generating-certificates-for-secure-tls-communication-between-ns-asjs
[EduRoam]: https://www.eduroam.org/
[LoRaWAN Backend Specifications]: https://lora-alliance.org/resource-hub/lorawanr-back-end-interfaces-v10
