```diff
+         This page is intended to be as a tutorial. It tries to provide some basic information about the Public Key
+         Infrastructure (PKI), X.509 Certificate and associated terms
+         It also provides informatin on how to generate the certificates and include them for an Open Roaming
+         scenario. 
```


## Certificates

Normally in a web communication, the Client (i.e. the web browser) will ask the web server that it is connecting to present a certificate as part of the initial connection set up. The certificate is needed for establishing a TLS (Transport Layer Security) communication over a Computer Network.  The client will validate the certificate by ascertaining:

1. The subject of the certificate matches the server name to which the client is trying to connect
2. The certificate is signed by a trusted certificate authority

The standard format for this certificate is called X.509. An X.509 certificate contains Public Key, An identity (e.g. Hostname or Organisation or Individual) and is either signed by a Certificate Authority (CA) or Self-Signed. 


## Certificate Authority (CA)

CA is a trusted third party that verifies the Public Key provided by the server and validates it by including  its digital signature into the X.509 Certificate. In layman terms, it is like the Government whch provides a digital signature for one's passport that can be verified and accepted in all Airports of the world. 


## Root Certificate

The root certificate, also called a trusted root, is one of the certificates issued by a trusted CA. It’s a special type of X.509 digital certificate which is used for issuing other certificates called intermediates and further end-user TLS Certificates for avoiding the risk of getting compromised.


## Intermediate Certificate

Issuing a digital Certificate to the end-user directly from the root certificate is too dangerous. The roots are extremely valuable. To further protect themselves the CAs, came up with another layer of security – the intermediate certificate.

The root CA signs the intermediate root with its private key, and in turn, the intermediate CA uses its private key to issue X.509 certificates to the general public. 


## Self-signed Certificate

A self signed certificate is a certificate signed by the same entity that the certificate verifies. It is like you approving your own passport application.


## Certificate Signing Request (CSR)

A CSR is a block of encoded text that is given to a Certificate Authority when applying for a TLS Certificate. It is usually generated on the server where the certificate will be installed and contains information that will be included in the certificate such as the organization name, common name (domain name), locality, and country. It also contains the public key that will be included in the certificate. A private key is usually created at the same time that you create the CSR, making a key pair. A CSR is generally encoded using ASN.1 according to the PKCS #10 specification.

The CA will use a CSR to create your TLS certificate, but it does not need your private key. You need to keep your private key secret. The certificate created with a particular CSR will only work with the private key that was generated with it. So if you lose the private key, the certificate will no longer work.

## Certificate Setup 

### Directory structure

This step is optional. But, it is better to have the directory structured like this to make it easy to look at!

```sh
        /home
            /certificates
                /config
                    ca-csr.json
		    config.json
		    intermediate-csr.json
		    /gateway-bridge
                    /application-server
		    	/api
		    		/server
					certifcate.json
				/client
					certificate.json       
                    /network-server
		    	/api
		    		/server
			     		certifcate.json
				/client
			     		certificate.json
                /certs                         
                    /ca				## In most cases, this directory will not be needed
                    	ca.csr
			ca-key.pem
			ca.pem
		    /intermediate               ## In most cases, Intermediate will be the CA
		    	intermediate.csr
			intermediate-key.pem
			intermediate.pem
		    /network-server
		    	/api
				/server
					network-server-api-server.csr
					network-server-api-server-key.pem
					network-server-api-server.pem
				/client
					application-server-api-client.csr
					application-server-api-client-key.pem
					application-server-api-client.pem
		    /application-server
		    	/api
				/server
				/client
```

### CA Certificate generation

#### Configuration

```diff
+         This section is needed only by the CA and might not be needed in most cases
```

Add to the respective folder, the file ```ca-csr.json```as in the [Directory Structure]
```sh
 		{
			 "CN": "Afnic CA",			# Cannonical Name
  			 "key": {
    					"algo": "rsa",
    					"size": 2048
  				},
			 "names": [
     			       {
               				"C": "FR",   		# Country
               				"L": "SQY",		# Location/City
               				"O": "Afnic",		# Organisation
               				"ST": "Yevlines"	# State
        			}
    				   ]
		}
```         
       
#### Generation

Now it is necessary to create the certificates for the CA following the commands below:

The following commands are run from the ```/certificates``` directory as in the [Directory Structure]

```sh
	mkdir -p certs/ca
        cfssl gencert -initca config/ca-csr.json | cfssljson -bare certs/ca/ca

``` 

The above command creates a new certificate, a key and a sign request in the ```/certs/ca``` directory as follows:
 * ca-key.pem (certificate key)
 * ca.pem (certificate)
 * ca.csr (sign request)
 
The ca.pem file is a public file, the ca.csr file is not too important as of now, as we won’t sign the request with another CA, while the ca-key.pem is your private key. This key is the object that you should keep safe. Keep it as safe as possible after you are done with this tutorial and do not share it with anyone. If anyone gains access to the CA, they can sign requests as if it was you doing it.

#### Verification

One should verify the generated keys with OpenSSL:

 * Verify the CSR file (At the beginning of the Output you should have text ```verify OK```)
 ```sh
	openssl req -text -noout -verify -in ca.csr
 ``` 
  
 * Verfiy the certificate (The output shows whether the certificate has been generated with your required inputs)
 ```sh
	openssl x509 -in ca.pem -text -noout
 ``` 
 * Verify the private key (At the beginning of the Output you should have text ```RSA key ok```)
  ```sh
	openssl rsa -in ca-key.pem -check
  ``` 

### Intermediate Certificate generation

#### Configuration

```diff
+       This section is needed by each Institution to generate certificates  for their AS or NS
```

The next steps require a profile config file ```config.json``` as in the [Directory Structure]. The profile describes general details about the certificate. For example it’s duration, and usages. We will use a single config file for all certificate generation. Thus, we set up a new “profile” for each of the entities concerned. In addition to the "Intermediate_Ca", we also add "Client" and "Server" profiles.

One can observe in the config file how the “client” profile specifies “client auth” in its usages, while the “server” profile specifies “server auth”, while 
the "intermediate-ca" profile does both.

Add to the respective folder, the file ```config.json```as in the [Directory Structure]
```sh
{
        "signing": {
                "default": {
                         "expiry": "8760h"
                          },
                          "profiles": {
                                   "intermediate_ca": {
                                            "usages": [
                                                       "signing",
                                                       "digital signature",
                                                       "key encipherment","cert sign",
                                                       "crl sign",
                                                       "server auth",
                                                       "client auth"
                                            ],
                                            "expiry": "8760h",
                                             "ca_constraint": {
                                                     "is_ca": true,
                                                     "max_path_len": 0,
                                                     "max_path_len_zero": true
                                              }
                                   },
                                   "client": {
                                                   "expiry": "8760h",
                                                    "usages": [
                                                               "signing",
                                                               "key encipherment",
                                                               "client auth"
                                                    ]
                                   },
                                   "server": {
                                                   "expiry": "8760h",
                                                   "usages": [
                                                               "signing",
                                                               "key encipherment",
                                                               "server auth"
                                                   ]
                                  }
                        }
          }
}
```         
Just as with the root, each intermediate certificate requires a CSR json file. Add to the respective folder, the file ```intermediate-csr.json```as in the [Directory Structure]  

```sh
{
  "CN": "Afnic Intermediate CA",
  "key": {
          "algo": "rsa",
          "size": 2048
         },
         "names": [
             {
                 "C": "FR",
                 "L": "SQY",
                "O": "Afnic-Labs",
                "ST": "Yevlines"
             }
        ],
        "ca": {
            "expiry": "42720h"
        }
}
```


#### Generation

Now it is necessary to create the certificates for the Intermediate CA following the commands below:

The following commands are run from the ```/certificates``` directory as in the [Directory Structure]

```sh
	mkdir -p certs/intermediate
        cfssl gencert -initca config/intermediate-csr.json | cfssljson -bare certs/intermediate/intermediate

``` 

The above command creates a new certificate, a key and a sign request in the ```/certs/ca``` directory as follows:
 * intermediate-key.pem (certificate key)
 * intermediate.pem (certificate)
 * intermediate.csr (sign request)
 
 And also outputs a logs as follows:
 
 ```sh
			[INFO] generate received request
			[INFO] received CSR
			[INFO] generating key: rsa-2048
			[INFO] encoded CSR
			[INFO] signed certificate with serial number 507554958619371358914352423324768250450624326809

 ``` 
#### Sign

Now we have to sign the ```intermediate CA```.  It is done by the command below which uses the CA produced previously to sign the intermediate CA. It also uses the “config.json” profile and specifies the “intermediate_ca” profile.

The following commands are run from the ```/certificates``` directory as in the [Directory Structure]

```sh
 cfssl sign -ca certs/ca/ca.pem -ca-key certs/ca/ca-key.pem -config config/config.json -profile intermediate_ca certs/intermediate/intermediate.csr |  cfssljson -bare certs/intermediate/intermediate
 ``` 

The above command should output something similiar as follows:

```sh
 [INFO] signed certificate with serial number 442522653809835936897657268932942954456988632585
 ``` 
 
#### Verification

One can verify the generated certificates using the commands in the [Verification] section


### Certificate generation at the NS

The NS will act as ```Client``` when trying to authenticate the AS and the NS will act as ```Server``` when the AS is trying to authenticate it. Both the communications for authentications are done by [API]. Hence there are two two groups of certificates needed:

   1. When the NS acts as a Client and 
   2. When the NS acts as the server


#### Client Configuration

Add to the respective folder under ```config/network-server/api/client```, the file ```certificate.json```as in the [Directory Structure]

```sh
	{
    		"CN": "6d5db27e-4ce2-4b2b-b5d7-91f069397978",
    		"hosts": ["127.0.0.1","localhost","192.168.1.4"],
    		"key": {
      		"algo": "rsa",
      		"size": 2048
    		}
	}
```  


```The certificate generated by this configuration file will be deployed in the web-interface when provisionning the ChirpStack Network Server.``` 

The CN field here corresponds to the unique identifier for the Application Server, accessible in the configuration file (.toml) as application_server.id. If it is not found one can add it.

In the "hosts" field, by default it is ["127.0.0.1","localhost"]. We added our Application server’s public IP here, ours is as follow : ["127.0.0.1","localhost","192.168.1.4"] )

       
#### Client Certificate Generation

Now it is necessary to create the certificates for the CA following the commands below:

The following commands are run from the ```/certificates``` directory as in the [Directory Structure]

```sh
	mkdir -p certs/network-server/api/client
	cfssl gencert -ca certs/intermediate/intermediate.pem -ca-key certs/intermediate/intermediate-key.pem -config config/config.json -profile client config/network-server/api/client/certificate.json | cfssljson -bare certs/network-server/api/client/network-server-api-client
``` 

The above command creates a new certificate, a key and a sign request in the ```certs/network-server/api/client``` directory as follows:
 * network-server-api-client-key.pem (certificate key)
 * network-server-api-client.pem (certificate)
 * network-server-api-client.csr (sign request)
 

#### Verification

One can verify the generated certificates using the commands in the [Verification] section


#### Server Configuration

Add to the respective folder under ```config/network-server/api/server```, the file ```certificate.json```as in the [Directory Structure]

```sh
	{
    		"CN": "Afnic-Network-Server",
    		"hosts": ["127.0.0.1","localhost","149.202.56.242"],
    		"key": {
      		"algo": "rsa",
      		"size": 2048
    		}
	}
```  


```The certificate generated by this configuration file will be deployed in the web-interface when provisionning the ChirpStack Network Server.``` 

The CN field could be any string. In the "hosts" field, by default it is ["127.0.0.1","localhost"]. We added our Network server’s public IP here, ours is as follow : ["127.0.0.1","localhost","192.168.1.3"] )

       
#### Server Certificate Generation

Now it is necessary to create the certificates for the CA following the commands below:

The following commands are run from the ```/certificates``` directory as in the [Directory Structure]

```sh
	mkdir -p certs/network-server/api/server
	cfssl gencert -ca certs/intermediate/intermediate.pem -ca-key certs/intermediate/intermediate-key.pem -config config/config.json -profile server config/network-server/api/server/certificate.json | cfssljson -bare certs/network-server/api/server/network-server-api-server
``` 

The above command creates a new certificate, a key and a sign request in the ```certs/network-server/api/server``` directory as follows:
 * network-server-api-server-key.pem (certificate key)
 * network-server-api-server.pem (certificate)
 * network-server-api-server.csr (sign request)
 

#### Verification

One can verify the generated certificates using the commands in the [Verification] section

=============================


### Certificate generation at the AS

The AS will act as ```Client``` when trying to authenticate the NS and the AS will act as ```Server``` when the NS is trying to authenticate it. Both the communications for authentications are done by [API]. Hence there are two groups of certificates needed:

   1. When the AS acts as a Client and 
   2. When the AS acts as the server


#### Client Configuration

Add to the respective folder under ```config/application-server/api/client```, the file ```certificate.json```as in the [Directory Structure]

```sh
	{
    		"CN": "000001",
    		"hosts": ["127.0.0.1","localhost","192.168.1.4"],
    		"key": {
      		"algo": "rsa",
      		"size": 2048
    		}
	}
```  


```The certificate generated by this configuration file will be deployed in the web-interface when provisionning the ChirpStack Network Server.``` 

The CN field here corresponds to the unique identifier for the Network Server, accessible in the configuration file (.toml) as network_server.net_id. If it is not found, one can add it.

In the "hosts" field, by default it is ["127.0.0.1","localhost"]. We added our Network server’s public IP here, ours is as follow : ["127.0.0.1","localhost","192.168.1.3"] )

       
#### Client Certificate Generation

Now it is necessary to create the certificates for the CA following the commands below:

The following commands are run from the ```/certificates``` directory as in the [Directory Structure]

```sh
	mkdir -p certs/application-server/api/client
	cfssl gencert -ca certs/intermediate/intermediate.pem -ca-key certs/intermediate/intermediate-key.pem -config config/config.json -profile client config/application-server/api/client/certificate.json | cfssljson -bare certs/application-server/api/client/application-server-api-client
``` 

The above command creates a new certificate, a key and a sign request in the ```certs/application-server/api/client``` directory as follows:
 * application-server-api-client-key.pem (certificate key)
 * application-server-api-client.pem (certificate)
 * application-server-api-client.csr (sign request)
 

#### Verification

One can verify the generated certificates using the commands in the [Verification] section


#### Server Configuration

Add to the respective folder under ```config/application-server/api/server```, the file ```certificate.json```as in the [Directory Structure]

```sh
	{
    		"CN": "Afnic-Application-Server",
    		"hosts": ["127.0.0.1","localhost","149.202.57.54"],
    		"key": {
      		"algo": "rsa",
      		"size": 2048
    		}
	}
```  


```The certificate generated by this configuration file will be deployed in the web-interface when provisionning the ChirpStack Network Server.``` 

The CN field could be any string. In the "hosts" field, by default it is ["127.0.0.1","localhost"]. We added our Application server’s public IP here, which is as follow : ["127.0.0.1","localhost","192.168.1.3"] )

       
#### Server Certificate Generation

Now it is necessary to create the certificates for the CA following the commands below:

The following commands are run from the ```/certificates``` directory as in the [Directory Structure]

```sh
	mkdir -p certs/application-server/api/server
	cfssl gencert -ca certs/intermediate/intermediate.pem -ca-key certs/intermediate/intermediate-key.pem -config config/config.json -profile server config/application-server/api/server/certificate.json | cfssljson -bare certs/application-server/api/server/application-server-api-server
``` 

The above command creates a new certificate, a key and a sign request in the ```certs/network-server/api/server``` directory as follows:
 * application-server-api-server-key.pem (certificate key)
 * application-server-api-server.pem (certificate)
 * application-server-api-server.csr (sign request)
 

#### Verification

One can verify the generated certificates using the commands in the [Verification] section



==============================



[Directory Structure]: #directory-structure
[Verification]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Certificates-Tutorial.md#verification
[API]: https://www.chirpstack.io/network-server/integrate/api/
