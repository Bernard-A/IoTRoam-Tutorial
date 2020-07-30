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

CA is a trusted third party that verifies the Public Key provided by the server and validates it by including  its digital signature into the X.509 Certificate. In layman terms, it is like the Government provides a authenticity for one's passport which is accepted in all Airports of the world. 


## Root Certificate

The root certificate, also called a trusted root, is one of the certificates issued by a trusted CA. It’s a special type of X.509 digital certificate which is used for issuing other certificates called intermediates and further end-user SSL Certificate for avoiding the risk of getting compromised.


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
            /chirpstack-certificates
                /config
                    ca-csr.json
		    intermediate-config.json
		    intermediate-csr.json
                    /chirpstack-gateway-bridge
                    /chirpstack-application-server
                    /chirpstack-network-server
                    /certs                         
                        /ca				## In most cases, this directory will not be needed
                    	/intermediate               	## In most cases, Intermediate will be the CA
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

The following commands are run from the ```/chirpstack-certificates``` directory as in the [Directory Structure]

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

The next steps require a profile config file. The profile describes general details about the certificate. For example it’s duration, and usages.


Add to the respective folder, the file ```intermediate-config.json```as in the [Directory Structure]
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
            							"key encipherment",
            							"cert sign",
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
      						   }
                             }
                           }
        }
```         
Add to the respective folder, the file ```intermediate-csr.json```as in the [Directory Structure]   
   
#### Generation

Now it is necessary to create the certificates for the CA following the commands below:

The following commands are run from the ```/chirpstack-certificates``` directory as in the [Directory Structure]

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





[Directory Structure]: #directory-structure
