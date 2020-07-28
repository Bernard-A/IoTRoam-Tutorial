This page provides an introduction to the Join Server (JS), JoinEUI and configuring the NS to enable DNS resolution. To communicate securely between the NS and AS this page also explains how to configure the TLS certificates:

 * [JS Brief Intro]
 * [Configuring NS to access the JS using DNS]
 * [Generating Certificates for secure TLS Communication between NS<->AS/JS]
 * [Deploying Certificates for secure TLS Communication between NS<->AS/JS]
 
 ## JS Brief Intro
 
JS is used for authenticating the end-devices, is also identified by a 64-bit globally unique identifier termed as AppEUI/JoinEUI. The JS could be a separate server or same as the AS. In our set up the JS and AS functionalities runs in the same physical machine as shown in the figure below:

<p align="center">
  <img width="760" height="300" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig14.png?raw=true">
</p>
 
 
 ## Configuring NS to access the JS using DNS

When the ED sends a Join Request (JR) as per the [LoRaWAN Backend Specifications], the RGW will forward the JR to the NS. If the ED is not known to the NS, then it generates a DNS query to securely communicate with the JS. 

In order to enable DNS resolution, the configuration file is located in the NS folder in ```/etc```. It is a ```toml``` file:

```sh
$ /etc/chirpstack-network-server/chirpstack-network-server.toml
```

Jump to the ```[join_server]``` section which is used to find the Join Server relevant to a given JoinEUI. The default configuration always uses the fallback method of using a default Join Server to resolve it (this Join Server being the Application Server). We will also setup the ```[join_server.default]``` parameter to contact our Application Server but we wish to enable the JoinEUI resolution. 

The configuration file already specifies the ```resolve_domain_suffix``` corresponding to the LoRa Alliance service for JoinEUI resolution, so we just have to change the resolve_join_eui Boolean to true to set things right:


```sh
[join_server]
.
.
resolve_join_eui=true
.
.
resolve_domain_suffix=”.joineuis.iotreg.net”
```
   
If your JS is not hosted on the same server as your NS, you will also need to change the ```[join_server.default]``` parameter with the IP address of your JS

```sh
[join_server]
.
.
   [join_server.default]
   .
   .  
   server=http://192.168.2.12:8003
   ```
   
 ## Generating Certificates for secure TLS Communication between NS<->AS/JS
 
The idea is to generate Certificates for both the NS and the AS. Follow the 7 steps as follows: 

1. For this platform set up, following the Chirpstack process, we need to install the [CFSSL] tool. This installation could be done in your local computer or the NS ot the AS:
```sh
 sudo apt-get install -y golang-cfssl
```
2. Clone the repository ```https://github.com/brocaar/chirpstack-certificates```
3. Fom the ```$ chirpstack-certificates``` directory

### 4. Configuration in the NS files 
 * Modify ```$ config/loraserver/api/server/certificate.json``` to fit your NS deployment
     * ```“CN”:”network-server-afnic``` => (You can put whatever you want in the place of ```network-server-afnic```)
     * ```"hosts": [YOUR_NETWORK_SERVER_HOST_IP_ADDRESSES]```. By default, it contains ```["127.0.0.1","localhost"]```. Our Configuration is updated as follows:
        ```sh
               ["127.0.0.1","localhost","vps***.ovh.net,1.2.3.4"] # We added our server’s domain name and public IP
        ```
 * Modify ```$ config/loraserver/api/client/certificate.json``` to fit your AS deployment
     * ```“CN”:“01234567-0123-0123-0123-0123456789ab”```  
        ```sh
        Copy the "ID" under the section [application_server] in the file  'chirpstack-application-server.toml'  
        in the application server in the place of '01234567-0123-0123-0123-0123456789ab' 
        ```

### 5. Configuration in the AS files 
 * Modify ```$ config/lora-app-server/api/server/certificate.json``` to fit your AS deployment        
     * ```“CN”:”app-server-afnic``` => (You can put whatever you want in the place of app-server-afnic)
     * "hosts": [YOUR_APPLICATION_SERVER_HOST_IP_ADDRESSES]``` => (by default : ["127.0.0.1","localhost"], we added in our server’s domain name and public IP here, ours is as follow : ["127.0.0.1","localhost","vps***.ovh.net,1.2.3.5"] )
 * Modify ```$ config/lora-app-server/api/client/certificate.json``` to fit your NS deployment
     * ``“CN”:”Copy the net_id under the section [network_server] in the file  ```chirpstack-network-server.toml ```  in the network server here" 
        
### 6. Configuration for the JS 
As mentioned earlier, We assume that the JS and the AS are in the same machine

 * Modify ```$ config/lora-app-server/join-api/server/certificate.json``` to fit your JS deployment (which is the AS also as per our set up)        
        * ```“CN”:”join-server-afnic``` => (You can put whatever you want in the place of join-server-afnic)
        * "hosts": [YOUR_APPLICATION_SERVER_HOST_IP_ADDRESSES]``` => (by default : ["127.0.0.1","localhost"], we added in our server’s domain name and public IP here, ours is as follow : ["127.0.0.1","localhost","vps***.ovh.net,1.2.3.4"] )
        * Add the JoinEUI's accepted by  the JS in the hosts section. Thus the hosts section would look as follows: 
        ```sh
              ["127.0.0.1","localhost","vps***.ovh.net,1.2.3.4","3.4.5.6.7.2.D.F.D.7.4.D.F.G.F.1.joineuis.iotreg.net"]
        ```
  * Modify ```$ config/lora-app-server/join-api/client/certificate.json``` to fit your NS deployment
        * ``“CN”:”Copy the net_id under the section [network_server] in the file  ```chirpstack-network-server.toml ```  in the network server here" 

7.	Run ```$ make``` from the ```$ chirpstack-certificates``` directory

The above command will generate certificates for the NS, AS and the JS. Each certificate configuration file generates 3 files (csr file, key.pem file and .pem file). The software also signs all certificate using a single ca accessible in the certs files when the program is run.

## Deploying Certificates for secure TLS Communication between NS<->AS/JS

One need to deploy each certificate in the respective servers configuration files.

### Configuring the NS with the certificates

 * Step 1: On the Network Server, one need to copy the  ```ca.pem ``` file, the  ```loraserver-api-server.pem ``` file and the  ```loraserver-api-server-key.pem ``` file to a specific directory. Then update the  ```[network_server.api] ``` section in the ```loraserver.toml``` file by adding the followingrequired lines : 
 ```sh
[network_server]
.
.
   [network_server.api]
   .
   .
   ca_cert="PATH_TO_CA/ca.pem"
   tls_cert="PATH_TO_TLS_CERT/loraserver-api-server.pem"
   tls_key="PATH_TO_TLS_KEY/loraserver-api-server-key.pem"
```
These are the certificates for the server-side of the API. These certificates are for securing the LoRa Server API which is by default listening on port 8000 

 * Step 2:  On the Network Server, one need to copy the  ```ca.pem ``` file,the ```lora-app-server-join-api-client.pem``` file and the ```lora-app-server-join-api-client-key.pem``` file to a specific directory. Then update the  ```[join_server.default] ``` section in the ```loraserver.toml``` file by adding the following required lines : 
 ```sh 
 [join_server]
.
.
.
   [join_server.default]
   .
   .
   .
   js_ca_cert="certs/ca/ca.pem"
   js_tls_cert="certs/lora-app-server/join-api/client/lora-app-server-join-api-client.pem"
   js_tls_key="certs/lora-app-server/join-api/client/lora-app-server-join-api-client-key.pem"
```
These are the client-side certificates (used by LoRa Server) to connect to the LoRa Server Join API.

 * Step 3: These are the client-side certificates (used by LoRa App Server) to connect to the LoRa Server API. One needs to copy-paste the content of the following files in the web interface:
    * CA certificate content of certs/ca/ca.pem
    * TLS certificate content of certs/loraserver/api/client/loraserver-api-client.pem
    * TLS key content of certs/loraserver/api/client/loraserver-api-client-key.pem
    
    To do this first connect to the NS web interface and set it up as shown in the figure below:
    
<p align="center">
  <img width="700" height="400" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig15.png?raw=true">
</p>
    
Then, add the three files mentioned in ```Step 3```  under *Certificates for LoRa App Server to LoRa Server connection* as shown in the figure below :
    
<p align="center">
  <img width="700" height="600" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig16.png?raw=true">
</p>
    
### Configuring the AS with the certificates

* Step 1: On the application Server, one need to copy the  ```ca.pem ``` file, the  ```lora-app-server-api-server.pem``` file and the  ```lora-app-server-api-server-key.pem``` file to a specific directory. Then update the  ```[network_server.api]``` section in the ```lora-app-server.toml``` file by adding the following required lines : 
 ```sh
[application_server]
.
.
   [application_server.api]
   .
   .
   ca_cert="certs/ca/ca.pem"
   tls_cert="certs/lora-app-server/api/server/lora-app-server-api-server.pem"
   tls_key="certs/lora-app-server/api/server/lora-app-server-api-server-key.pem" 
```
These certificates are for securing the LoRa App Server API which is by default listening on port 8001 (see lora-app-server.toml). This is not the REST API!

 * Step 2:  On the Network Server, one need to copy the  ```ca.pem ``` file,the ```lora-app-server-join-api-client.pem``` file and the ```lora-app-server-join-api-client-key.pem``` file to a specific directory. Then update the  ```[join_server.default] ``` section in the ```loraserver.toml``` file by adding the following required lines : 
 ```sh 
 [join_server]
.
.
js_ca_cert="certs/ca/ca.pem"
js_tls_cert="certs/lora-app-server/join-api/server/lora-app-server-join-api-server.pem"
js_tls_key="certs/lora-app-server/join-api/server/lora-app-server-join-api-server-key.pem"
```
These certificates are for securing the LoRa App Server Join API which is by default listening on port 8003

 * Step 3: These are the client-side certificates (used by LoRa Server) to connect to the LoRa App Server API. One needs to copy-paste the content of the following files in the web interface:
    * CA certificate content of certs/ca/ca.pem
    * TLS certificate content of certs/lora-app-server/api/client/lora-app-server-api-client.pem
    * TLS key content of certs/lora-app-server/api/client/lora-app-server-api-client-key.pem

Then, add the three files mentioned in ```Step 3```  under *Certificates for LoRa Server to LoRa App Server connection* using the web interface as indicated above.

[LoRaWAN Backend Specifications]: https://lora-alliance.org/resource-hub/lorawanr-back-end-interfaces-v10
[Configuring NS to access the JS using DNS]: #configuring-ns-to-access-the-js-using-dns
[JS Brief Intro]: #js-brief-intro
[Generating Certificates for secure TLS Communication between NS<->AS/JS]: #generating-certificates-for-secure-tls-communication-between-ns-asjs
[Deploying Certificates for secure TLS Communication between NS<->AS/JS]: #deploying-certificates-for-secure-tls-communication-between-ns-asjs
[CFSSL]: https://cfssl.org/
