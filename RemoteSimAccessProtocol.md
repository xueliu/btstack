# Introduction #

[From Wikipedia](http://en.wikipedia.org/wiki/Bluetooth_profile#SIM_Access_Profile_.28SAP.2C_SIM.2C_rSAP.29): "The  Remote Sim Access Protocol ... allows devices such as car phones with built in GSM  transceivers to connect to a SIM card in a phone with Bluetooth, thus the car phone itself doesn't require a separate SIM card. This profile is also known as rSAP (remote-SIM-Access-Profile).."

So far, I haven't seen a specification for that but as far as I understand that summary, it allows the car mobile telephone to exchanged data with the SIM card over Bluetooth. So, there are two main component: Bluetooth communication and SIM access.

# Details #

The Bluetooth communication is detailed in the spec below. However, I don't know, how an application on the iPhone can "talk" to the "SIM-Card". There is a /dev/tty.umts, but I have no idea what to do with it. It looks like the "at+csim=..." command is used on some mobile phone chips.

If anybody is interested in looking into this, or has more pointers, please add comments or join the BTstack Google Group.

# Spec #
I'm certain that the specification was missing at Bluetooth.com, but it's there now
[Sim Access Procotol Specification v1.1](http://www.bluetooth.com/Specification%20Documents/SAP_SPEC_V11.pdf)

# Random bits #
  * [http://wiki.openmoko.org/wiki/Hardware:AT_Commands AT Commands used by the OpenMoko Freeruner) -> looks like AT+CSIM is used to access SIM card, other links there
  * http://lists.openmoko.org/pipermail/gsmd-devel/2007-March/000009.html
  * [Implementation for Symbian](http://developer.symbian.org/oss/MCL/sf/mw/btservices/file/a42ed326b458/bluetoothengine/btsap/)