```diff
-           In this document, We have used Multitech products as ED. If you are using a different
-           Hardware you have to follow those Hardware specific details to set up the ED.
+           Please follow the Pointer section below for further details in this case
```

# Multitech End-Device Setup

This page explains setting up of the ED, so that it can send data over LoRa Radio Frequency.

## Hardware Setup

We are using [Multitech mDoT]. The mDot is fixed to the [Micro Developer Kit]. One has to connect the LoRa Antenna to mDot. 

<p align="center">
  <img width="560" height="400" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig2.png?raw=true">
</p>

The Micro Developer Kit where the ED is fixed enables one to access the ED from one's computer. As shown in the Figure 1, the Micro Developer Kit is connected with a USB connection which can be connected to one's computer. 

 ``` If one doesn't have an USB connector like in Figure 1, one can also directly connect the Micro Developer Kit to the Computer's USB port. ```

Before your Connect the other end of the USB connector to the computer, if you are using a Linux Distribution, type ```ls /dev/tty*``` in your terminal. Have a look at the existing tty serial connections.

Now, connect the other end of the USB connector the computer, you will see red, blue LEDs in the micro developer kit. Now type ```ls /dev/tty*``` in your terminal, you will see two new serial connections. Normally they will be of form ```/dev/tty/ACM0, /dev/tty/ACM1``` or ```/dev/ttyXRUSB0, /dev/tty/ACM0```.

*If the mDoT is not updated with the latest firmware, it needs to be [downloaded] to your computer*

Of the two serial connections, one is for ```DEBUG``` and another is for ```Communicating with the device```.

Use the [Serial console]<sup>1</sup>. Launch two serial consoles for both serial connections assuming the serial connections are ``` ACM0 and ACM1 ```

 ```sh
screen /dev/ttyACM0 115200
screen /dev/ttyACM1 115200
  ```

The DEBUG screen terminal is the one that shows debug commands such as in Figure 2

<p align="center">
  <img width="300" height="100" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig3.png?raw=true">
</p>

In the other screen terminal, you should be able to type the command ```at``` and get an ```Ok``` as shown in the Figure 3. This tells that you have control over the LoRa end-device. 

<p align="center">
  <img width="150" height="100" src="https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Images/Fig4.png?raw=true">
</p>

## Pointer Section

In the event of one using ED that is not Multitech mDoT, please make sure that you are able to access the device. Once verified, [Next section to follow] 

If you are using Multitech mDot and you have verified accessing the ED by following the page, [Next section to follow] 

In any case, If you want to go to the [previous page]

## Footnotes
[1] The example to install screen is for Fedora. You install ‘screen’ if you don’t have according to your Linux distro requirements

[Multitech mDoT]: https://www.multitech.com/brands/multiconnect-mdot
[Micro Developer Kit]: https://www.multitech.com/brands/micro-mdot-devkit
[Serial console]: https://tinyurl.com/uxtkgt2
[Next section to follow]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Gateway-Setup.md
[previous page]: https://github.com/sandoche2k/IoTRoam-Tutorial/blob/master/Architecture.md
[downloaded]: https://www.multitech.net/developer/downloads/
