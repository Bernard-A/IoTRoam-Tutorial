This page provides an introduction to the Join Server (JS), JoinEUI and configuring the NS to enable DNS resolution. To communicate securely between the NS and AS this page also explains how to configure the TLS certificates:

 * [JS Brief Intro]
 * [Configuring NS to enable DNS resolution]
 * [Adding Certificates for secure TLS Communication between NS<->AS]
 
 ## JS Brief Intro
 
JS is used for authenticating the end-devices, is also identified by a 64-bit globally unique identifier termed as AppEUI/JoinEUI. The JS could be a separate server or same as the AS. In our set up the JS and AS functionalities runs in the same physical machine as shon in the figure below:

<p align="center">
  <img width="760" height="300" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig14.png?raw=true">
</p>
 
 
 ## Configuring NS to enable DNS resolution

The configuration file is located in the NS folder in ```/etc```. It is a ```toml``` file:

```sh
$ /etc/chirpstack-network-server/chirpstack-network-server.toml
```

Jump to the ```[join_server]``` section which is used to find the Join Server relevant to a given JoinEUI. The default configuration always uses the fallback method of using a default Join Server to resolve it (this Join Server being the Application Server). We will also setup the ```[join_server.default]``` parameter to contact our Application Server but we wish to enable the JoinEUI resolution. 

The configuration file already specifies the resolve_domain_suffix corresponding to the LoRa Alliance service for JoinEUI resolution, so we just have to change the resolve_join_eui Boolean to true to set things right:


```sh
[join_server]
.
.
resolve_join_eui=true
.
.
resolve_domain_suffix=”.joineuis.iotreg.net”
```
   
If your AS is not hosted on the same server as your NS, you will also need to change the ```[join_server.default.server]``` parameter with the IP address of your AS

```sh
[join_server]
.
.
   [join_server.default]
   .
   .  
   server=http://10.1.86.48:8003
   ```
   
 ## Adding Certificates for secure TLS Communication between NS<->AS
 
The idea is to generate Certificates for both the NS and the AS. For this platform set up, following the Chirpstack process, we need to install the CFSSL tool
 
 
[Configuring NS to enable DNS resolution]: #configuring-ns-to-enable-dns-resolution
[JS Brief Intro]: #js-brief-intro
[Adding Certificates for secure TLS Communication between NS<->AS]: #adding-certificates-for-secure-tls-communication-between-ns-as
[CFSSL]: https://cfssl.org/
