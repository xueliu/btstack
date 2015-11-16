# Introduction #

After RS232 is gone for good, common SmartCard readers are connected via USB and follow the [USB CCID specification](http://www.usb.org/developers/devclass_docs/DWG_Smart-Card_CCID_Rev110.pdf).

An open source implementation of the CCID protocol is available as part of the [PCSC/lite project](http://pcsclite.alioth.debian.org/ccid.html).

However, there is no Bluetooth specification for SmartCard readers by the Bluetooth SIG.

In theory, SmartCard vendors could follow a similar approach as with USB Human Interface Devices (HID), where the 4 USB HID endpoints are mapped to 2 L2CAP channels. Another approach, found with the [HID OMNIKEY 2061 Bluetooth Reader](http://www.hidglobal.com/prod_detail.php?prod_id=369), is to put the TPDUs defined by the CCID specification into a minimal packet and send that over RFCOMM.

# Details for individual readers #

## [HID OMNIKEY 2061 Bluetooth Reader](http://www.hidglobal.com/prod_detail.php?prod_id=369) ##

After pairing with the default PIN '0000', which could be changed according to the manual, the reader provides an RFCOMM service at channel #1. Over this channel CCID TPDUs can be send. The TPDUs are wrapped in a minimal packet consisting of the start symbol 0xa5 and an XOR checksum over the TPDU.
Responses from the reader are wrapped by the same convention.

Example - the TPDU to turn power on for the first card slot is
```
62 00 00 00 00 00 00 00 00 00
```

To send it over Bluetooth to the OMNIKEY 2061, you would send:
```
A5 | 62 00 00 00 00 00 00 00 00 00 | 62
```

Note the 0x62 at the end, which was easy to calculate in this case :)


