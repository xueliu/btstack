# Introduction #

The 2.x OS on the iPhone and the iPod touch did only support Bluetooth headsets. However, this is not a limitation of the used Bluetooth chipset, but of an incomplete Bluetooth stack.

OS 3.0 provides support for headsets (mono/stereo), networking via the PAN/BNEP protocol and support for "Made for iPhone" devices which require a proprietary Bluetooth protocol based on top of SPP. Still, not even basic OBEX support is included (available with any low-price mobile phone), and no API is available to connect to arbitrary Bluetooth devices. It also extents the GameKit API to allow for iOS-to-iOS communication.

OS 4.0 added support for A2DP and braille readers over RFCOMM.

OS 5.0 on the new iPhone 4S added support for Bluetooth Low Energy device by the public CoreBluetooth framework (also available on 10.7.2 with appropriate Bluetooth 4.0 modules, e.g. in the 2011 series of MacBook (Pro) and Mac mini)

OS 6.0 added support for Message Access Profile (MAP) used in some cars. CoreBluetooth gains the ability to work in Peripheral mode (in addition to Central) mode.

OS 7.0 added AirDrop which uses Bluetooth Low Energy to discover other devices, but uses Apple's variant of Wifi direct for the actual data transfer.

## Bluetooth Accessories - Made for iPhone ##
Apple's External Accessory API introduced in 3.0 allows to register for external accessories which can be connected either via the Dock connector or via Bluetooth. It then allows to specify an accessory based on a so-called "protocol" string and provides a socket-like connection to the device. Almost 2 years later, devices using MfI appear on the market, e.g. the UnityRemote by Gear4.com.

To use this API, you'll have to sign up and get accepted into the "Made for iPhone" (MfI) program, and sign an NDA (or more...). If you get accepted into Mfi, you can purchase a MfI developer board. Such boards are currently provided by:
  * [Roving Networks, Inc](http://www.rovingnetworks.com/Apple_iOS_Support)
  * [LinTech GmbH](http://lintech.de)
  * [connectBlue](http://www.connectblue.com/nc/press/news-archive/news-single/artikel/press-release-connectblueTM-releases-bluetooth-io-module-with-iphoneandroid-and-analog-io-suppor/?cHash=5e9a3176a4cff1de041cb1b194fce29c)
  * [BlueGiga](http://groups.google.com/group/free-online-advertising-group/browse_thread/thread/0e49ae15a2014b5d/8c38144d16a6b236?show_docid=8c38144d16a6b236)

From the limited information available, it is reasonable to assume that Apple uses their iPod Accessory Protocol, which uses a basic UART on the iPod dock connector, over Bluetooth. Both make use of a  proprietary "authentication chip" which is exclusively sold by Apple.

## Peer-to-Peer Gaming ##
Apple's GameKit API in 3.0 allows to connect nearby iPhones via Bluetooth and to use normal IP networking (TCP, UDP, Bonjour..). The main goal of this is to simplify the pairing procedure which does not involve PINs. It makes use of the Extended Inquiry procedure of Bluetooth 2.1 and higher. The extended inquiry response (EIR) contains a list of SDP service records of which one is a "vendor-specific" extension. If a device provides such a EIR, the iPhone will connect to the foreign device and query its SDP for more details. More analysis is needed to provide peer-to-peer gaming for non-Apple devices.

## Bluetooth Tethering ##
On 3.0, the iPhone is provides Bluetooth PAN/BNEP support. If the devices are paired and Bluetooth is enabled on the iPhone,  a network connection can directly be established from another device, e.g., a laptop. Internet Tethering will activated automatically. The iPhone acts as a normal home router and allows direct TCP/IP connections in both directions. The use of standard TCP/IP communication might be sufficient for most connections between networked devices like netbooks and laptops and the iPhone. However,  connecting to standard Bluetooth devices is not possible by this.

## Bluetooth Low Energy via CoreBluetooth ##
On devices with Bluetooth 4.0, e.g. the new iPhone 4S, the public CoreBluetooth framework is available.

From Apple's docs:

> The Core Bluetooth framework (CoreBluetooth.framework) allows developers to
> interact specifically with Bluetooth Low-Energy ("LE") accessories. The Objective-C
> interfaces of this framework allow you to scan for LE accessories, connect and
> disconnect to ones you find, read and write attributes within a service, register
> for service and attribute change notifications, and much more.

> For more information about the interfaces of the Core Bluetooth framework,
> see the header files.

In iOS 5.0, CoreBluetooth only supports the Central role in which it can connect to an accessory.

In iOS 6.0, support for the Peripheral role was added. By this, an iPhone 4S or iPad 3 can act as accessory (e.g. pretend to be a heart rate monitor).

# Hardware Details #
Both generation of the iPhone (2G and 3G versions) use Bluetooth chipsets from Cambridge Silicon Radio (CSR), one of the leading Bluetooth vendors. CSR provides a wealth of documentation in their [technical support area](http://csrsupport.com) (free login required). The first generation iPhone contains a BlueCore 4, the second generation (current 3G models) contains a BlueCore 6 chipset. All newer devices, from the iPod touch second generation and the new iPhone 3GS on, contain Broadcom chipsets, a company not famous for providing developer information.

The Bluetooth modules are connected via an UART integrated in the ARM CPU. It is accessibly as /dev/tty.bluetooth. There are 2 virtual devices: /dev/btwake and /dev/btreset which probably are used to simply wake or reset the Bluetooth module via POSIX API instead of using some obscure internal library.

# Software Details #

On the iPhone and the iPod touch, the Bluetooth functionality is provided by BTserver, a background daemon. No further information is available about it. However, to control the Bluetooth chipset, the BTserver makes use of the BlueTool. The BlueTool tool provides a command-line interface for changing the operation mode (on/off/sleep..) and to configure various details of the Bluetooth chipset. The configuration used on the iPhone is rather complex, but fortunately, Apple provides commented initialization scripts in /etc/bluetool. By just feeding the script for the actual Bluetooth chipset to BlueTool, Bluetooth is fully initialized. The iPhones are configured for the [H5 transport protocol](HCI_UART_Transport.md) and a baudrate of 2.4 megabit per second. On the iPod touch, the firmware for the Broadcom chipset is loaded first before it is also configured for 2.4 mbps. In contrast to the iPhone, the basic H4 transport protocol is used. As part of the init script the the Bluetooth MAC address is set by the BlueTool as the initial one in the CSR chipsets is differs to the one reported by the About panel, and is zero for the Broadcom chipset. From OS 4.x, BlueTool has the configuration scripts compiled in.

# BTstack on the iPhone / iPod touch #

## Support for the integrated Bluetooth chipsets ##
So far, BTstack only support the [H4 transport mode](HCI_UART_Transport.md). Support for the [H5](HCI_UART_Transport.md) or the [CSR BCSP protocol](HCI_UART_Transport.md) might be implemented later. In order to use the CSR module in H4 mode, the script provided by Apple is parsed and modified on-the-fly before it is fed into BlueTool. Baud rates from 57600-921600 can be used. Higher baud rates are not supported by the standard IOCTL call.

## launchd integration ##
For a resource-limited device as the iPhone, the BTstack resp. the BTdaemon should only run, when Bluetooth functionality is needed/used by a client application. For Mac OS X, Apple recommends the use of Unix Domain sockets for IPC in general and to use launchd to start background daemons such as the BTdaemon on demand in their  [http://developer.apple.com/mac/library/technotes/tn2005/tn2083.html Technical Note TN2083
Daemons and Agents]. A nice introduction is also given in the [Google Tech Talk on launchd by its creator Dave Zarzycki](http://video.google.com/videoplay?docid=1781045834610400422). BTstack can be compiled with launchd support.

## Bluetooth Icon in Springboard Status ##
A background daemon cannot set an icon in the status bar (AFAIK). The new SpringBoardAccess MobileSubstrate extension hooks into the SpringBoard and allows to control the icons from anywhere. It might also be used to pop-up a message like "connection lost"

## Apple's Bluetooth Stack ##
Only one Bluetooth Stack can run at the same time. When BTdaemon is asked to enable Bluetooth, it tries to access the Bluetooth resource. If it cannot access the Bluetooth module, an error is reported back to the client, which in turn can infrom the user.  After the user turns off the original Bluetooth stack, a BTstack-based app can be started again. Later, we could add the option to display a warning and disable the original stack automatically.