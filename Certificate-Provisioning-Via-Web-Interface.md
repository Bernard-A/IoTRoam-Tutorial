# Provisioning the Client Certificates for the NS and the AS via the Web-Interface

The Server side certificates are configured via the respective `.toml`files. For the NS and the AS, the Client-end certificates are configured via the Web interface following the different steps:
   * Login (https://www.chirpstack.io/application-server/use/login/) to the Web interface
   * Make sure that the initial steps of configuring the web interface are completed (https://github.com/afnic/IoTRoam-Tutorial/blob/master/ApplicationServer-Setup.md#web-interface-setup)
   * Further, click on the "NS" menu as shown in the Figure 25.

<p align="center">
  <img width="760" height="200" src="https://github.com/afnic/IoTRoam-Tutorial/blob/master/Images/Fig25.png?raw=true">
</p>

## Provisioning the Client Certificates for the NS

On clicking the NS name (e.g. `Afnic-NS`as shown in Figure 25), one should click on the `TLS Certificates` tab  as shown in Figure 26.

<p align="center">
  <img width="700" height="300" src="https://github.com/afnic/IoTRoam-Tutorial/blob/master/Images/Fig26.png?raw=true">
</p>

One must copy the content of the following files under `Certificates for ChirpStack Application Server to ChirpStack connection`:

   * *CA certificate* content of `certs/ca/ca.pem`
   * *TLS certificate* content of `certs/network-server/api/client/network-server-api-client-combined.pem`
   * *TLS key* content of `certs/network-server/api/client/network-server-api-client-key.pem`


## Provisioning the Client Certificates for the AS

Further down in the same page; One must copy the content of the following files under `Certificates for ChirpStack to ChirpStack Application Server connection`as shown in Figure 27:

   * *CA certificate* content of `certs/ca/ca.pem`
   * *TLS certificate* content of `certs/network-server/api/client/application-server-api-client-combined.pem`
   * *TLS key* content of `certs/network-server/api/client/application-server-api-client-key.pem`



<p align="center">
  <img width="700" height="300" src="https://github.com/afnic/IoTRoam-Tutorial/blob/master/Images/Fig27.png?raw=true">
</p>
