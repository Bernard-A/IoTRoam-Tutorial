# Multitech End-Device Setup

## Hardware Setup

We are using Multitech mDoT (https://www.multitech.com/brands/multiconnect-mdot). The mDot is fixed to the Micro Developer Kit (https://www.multitech.com/brands/micro-mdot-devkit). One has to connect the LoRa Antenna to mDot. Then connect a USB wire to the Micro developer Kit
as shown in the Figure 1:

![alt text](https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig2.png?raw=true)

Before your Connect the other end of the USB connector to the computer and if you are using a Linux Distribution, type ```ls /dev/tty*``` in your terminal. Have a look at the existing tty serial connections.

Now, connect the other end of the USB connector the computer, you will see red, blue LEDs in the micro developer kit. Now type ```ls /dev/tty*``` in your terminal, you will see two new serial connections. Normally they will be of form ```/dev/tty/ACM0, /dev/tty/ACM1``` or ```/dev/ttyXRUSB0, /dev/tty/ACM0```.

Of the two serial connections, one is for “DEBUG” and another is for “Communicating with the device”.

Use the Serial console (https://tinyurl.com/uxtkgt2<sup>1</sup>. ). Launch two serial consoles for both serial connections
 ```sh
screen /dev/ttyACM0 115200
screen /dev/ttyACM1 115200
  ```

The DEBUG screen terminal is the one that shows debug commands such as in Figure 2



## Footnotes
[1) The example here is for Fedora. You install ‘screen’ if you don’t have according to your Linux distro requirements
