## <font color='red'><b>NEW!</b> Get the documentation for embedded systems: <a href='http://bluekitchen-gmbh.com/docs/btstack-gettingstarted-1.0.pdf'>BTstack Manual v1.0</a>.</font> ##


BTstack reached version 0.5 and allows for incoming and outgoing L2CAP and RFCOMM connections, registration of SDP records and to provide an ATT server. It supports Bluetooth Human Interface Devices via L2CAP (keyboard, WiiMote, etc.. ), the control of robots, and basic RFCOMM-based devices like standard GPS receivers, the [ZeeMote JS-1 joystick](http://www.zeemote.com/), or a Bluetooth Chipcard reader.

It runs in a single thread in user-space and makes use of the standard POSIX API. It expects a Bluetooth module connected via a serial connection (Bluetooth H4 HCI Transport). Support for Bluetooth USB modules is not working yet.

In particular, it runs on all iPhones and all iPod touch devices with Bluetooth support and iOS from SDK 3.0. On this platform, BTstack is automatically started by the system's launch daemon and provides its own status bar icon to distinguish itself from Apple's Bluetooth stack. It is available as binary via the [Cydia distribution system](http://www.saurik.com/id/1).


# Bluetooth Layers #

## HCI ##
  * A minimal HCI layer is implemented. Instead of implementing each HCI command separately, a table-driven approach allows to add HCI commands by only specifying the command packet format.
  * HCI Transport:
    * H4: The original H4 protocol for UARTs is fully implemented and working.
    * BCSP/H5: The more advanced CSR BSCP and the "Three Wire UART" transport protocol are not supported.
    * USB: Some code exists, but not working yet.

## [L2CAP](L2CAP.md) ##
  * The L2CAP State Machine can open an HCI connection and set up an outgoing L2CAP channel. It also supports incoming L2CAP channels by registering a predefined PSM. Incoming ACL packets are reassembled before delivered to the client application. More work has to done: MTU negotiation, packet segmentation, and flow control are not implemented.

## [SDP](SDP.md) ##
  * SDP Server support is basically working and almost complete. No documentation or examples yet.
  * SPD Client functionality is not directly provided, but sdp\_util.c can help to create  ServiceSearchPattern, AttributeIDList and process received AttributeLists.

## [RFCOMM](RFCOMM.md) ##
  * RFCOMM client and server functionality was released May 2011.

# Other #

## iPhone and iPod touch ##
  * The iPhone control component is able to switch the Bluetooth chipset on/off and properly initialize them in HCI H4 Transport mode with baud rates from 57600-921600 on all iPhone and iPod touch devices with build-in Bluetooth. On newer devices with Broadcom Chipsets and iOS 4.0+, the native speed of 3 mbps is used.
  * BTdaemon is automatically started by launchd. This makes it more robust and allows for automatic power-off, when no application is using Bluetooth.
  * The SpringBoardAccess MobileSubstrate extension allows the BTdaemon to show a BTstack icon similar to Apple's Bluetooth icon in the status bar.
  * Working on all iDevices that have a build-in Bluetooth module. It also works on iPod touch 1st generation if an external HCI H4 Bluetooth module is connected.

## Logging/Debugging ##
  * hci\_dump.c supports multiple formats:
    * Apple's PacketLogger
    * Linux BlueZ format
    * Direct packet hexdump to stdout.

## Integration with existing Run Loop ##
The BTstack and its client library can work with different run loops. BTdaemon uses its own basic select()-based run loop that supports POSIX file descriptors and timers.The client library can used with the POSIX run loop as well as with a Cocoa CFRunLoop wrapper inside of an Cocao(Touch) UI application.