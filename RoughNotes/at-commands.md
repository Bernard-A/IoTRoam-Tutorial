


## Configure mdot for the things network (Serial port)

```sh
        ATI                                       ## To get the current firmware version
        AT+NJM=1                                  ## Configure mDot for OTA join mode (default)
        AT+DI                                     ## To get the DEvEUI of the device
        AT+NI=0,008000000000d2c3                  ## Include Your App EUI
        AT+NK=0,1df708a3e7374cde7d66a10f70943277  ## Include Your App Key
        AT+NJM=1                                  ## Configure mDot for OTA join mode (default)
        AT+JOIN                                   ## Send a join request to the server
        AT+NJS                                    ## Display current join status 0:not joined, 1:joined
        AT&F                                      ## To set to Factory defaults
        AT&W                                      ## Save your changes
        AT&V                                      ##  Display current settings and status
```
