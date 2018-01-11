---
title: Micromax/Qualcomm modem interception, emulating network selection on linux
date: 2011-08-22 15:10:00.000000000 +05:30
---

I got this hsia wcdma modem with my EVDO connection, it is a buggy hardware, the firmware is buggy, the device seems to be using Qualcomm chipset,  vid = 0x1e0e, pid = 0xcefe. On forcing modem to HSIA modem, the modem switches to CDMA network mode,  Further the drivers that comes with device, a driver to eject the emulated usb cd rom, a usb-serial driver, The driver limits the speed of a single network(TCP,UDP...) connection to maximum of 32 kbps, multi connection downloaders works great.

On linux situation was little better but not different, if you used usbserial module, the speed of the interface is limited to 512kbps, no matter what was baud rate, data flow controls, etc. Further I wanted to emulate, the mode switching stuff on linux too. I intercepted the connection between the modem and the modem dialer from micromax. Here's the intercepted data. The data is commented wherever I felt to.

Intercepted data between Modem and proprietary software, pastebin, <a href="http://pastebin.com/CJAQQMuu" title="Intercepted data between Modem and it's software">Raw data</a>

## Mode changing using wvdial

The modem application was sending "AT^PREFMODE=2" to change to CDMA network, "AT^PREFMODE=4" to change to HSIA network, "AT^PREFMODE=8" for auto selection of network, as I mentioned the device firmware is buggy, if you force him to select HSIA network, it selects CDMA network. You can use these non-standard AT commands as Initializing strings in wvdial. For example

```
[Dialer Defaults]
Init1 = ATZ
Init2 = AT^PREFMODE?
Init3 = AT^PREFMODE=8
Stupid Mode = yes
Modem Type = Analog Modem
New PPPD = yes
Software Flow = no
Phone = #777
Modem = /dev/ttyUSB2
Username = ----------
Password = ----------
Baud = 9600
```

## Data flow control in newer wvdial

Option Software flow is not part of standard wvdial distributions, It's added by me, check my git repository for updated version <a href="https://github.com/linuxexp/wvdial">here</a>

You may want to use "AT+CNSMOD=1" to re-register on your network, "AT+AUTOCSQ=1" to make modem send signal values automatically after certain interval.For your convenience you can create a another section in wvdial.conf for such things. Here's an example

```
[Dialer Defaults]
Init1 = ATZ
Init2 = AT^PREFMODE?
Init3 = AT^PREFMODE=8
Stupid Mode = yes
Modem Type = Analog Modem
New PPPD = yes
Software Flow = no
Phone = #777
Modem = /dev/ttyUSB2
Username = ----------
Password = ----------
Baud = 9600 

[Dialer signal]
Init1 = ATZ
Init2 = AT+AUTOCSQ=1
Phone = #777
Modem = /dev/ttyUSB1
```

To dial with signal section, in terminal type `sudo wvdial signal`. The signal values returned is in following format

```
+CSQ: [signal_value], [bit_error_rate]
[signal_value]:
0: -113 dBm or less
1:-111 dBm
30:-109 to –53 dBm
31:-51dBm or greater
99:not known or not detectable 
```

## Linux kernel 2.6.38 updates
Linux kernel 2.6.38 comes with few updates in this regard, first the devices is automatically switched to Modem mode, second it comes with options module designed for HSIA modems, which works on top of usbserial, so you'll get your real speed under linux.

## Running on custom linux systems

### Kernel configuration

Now if you are on a custom linux or some extremely scaled down distro. what should you do?

You should first must have support for networking, dialup , USB devices, serial communication over usb in the kernel you have, most will have unless it's something for basic embedded device, options module most probably won't be there in these scaled down linux distros eg. puppy linux, tinycore, microcore doesn't have it.

Next you should have options modules, you can either have the module build in or externally loadable. if you go with later you must have support for loading external kernel modules, options module depends upon usbserial module, so you must add that too and its dependencies. if you compiling kernel &lt; 2.6.38 then you may or may not have the options module, not a problem usbserial will work, but there may be speed issues like I previously mentioned.

Here's the basic idea, how things should work, first when you plugin the device may or may not be detected, you may have to force the driver on it. Assuming that the device is detected as CDROM you now have to send SCSI eject commad to the cdrom, this can be done with <code>eject</code> command, like <code>eject /dev/cdrom</code> this will change the device mode from cdrom to modem, here if you have right things done the device may be detected as a modem, you can confirm this by listing /dev/ttyUSB* if you see something then device is detected otherwise you have to force module on device..

#### Forcing usbserial on device, when compiled to be in-build module
if you compiled usbserial as inbuild module in kernel, you can force the module to load for particular device like this, you have to add the following line as a kernel boot parameter, append the following line at the end of kernel boot line (mostly on grub) in your boot config file, You may want to check with your boot program's documentation on how to pass parameters to kernel.

```
usbserial.vendor=0x1e0e usbserial.product=0×cefe 
```

You can get VID and PID of your device on linux by issuing `lsusb` after you switch the device to MODEM mode from CDROM

#### Forcing usbserial on device, when compiled as external loadable module

if you want to force loading of usbserial module compiled as externally loadable kernel module, you can do so by using modprobe

```
modprobe usbserial vendor=0x1e0e product=0×cefe
```

You must be root to do that

The device should be ready by now, you can verify this by listing `/dev/ttyUSB*` if you see something then device is ready otherwise you can check for error by `dmesg` command.