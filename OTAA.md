This page covers setting up the NS to enable DNS resolution:

 * [Configuring NS to enable DNS resolution]
 
 # Configuring NS to enable DNS resolution

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
 
 [Configuring NS to enable DNS resolution]: #network-server-setup
