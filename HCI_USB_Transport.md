# Introduction #
As serial Bluetooth modules are a not commonly used, using a USB Bluetooth Dongle helps developing the upper layers of the stack. For BTstack, we have started to use [libUSB-1.0](http://libusb.wiki.sourceforge.net/Libusb1.0). It is available for Linux and OS X and provides an asynchronous API that facilitates integration in the BTdaemon single-threaded run-loop (see [Architecture](Architecture.md)).

However, it is not ported to Windows yet. For debugging purposes on Windows, a separate thread could use the older libUSB-0.1 and signal the main loop when data is ready to read from USB. According to Michael Murphy, there is recent discussion on the libusb-win32-dev mailing list about porting libusb-1.0 to Windows. Daniel Drake, the maintainer of the code, has pointed out that the POSIX fd polling used heavily in libusb-1.00 but absent in Windows, is not immutable code, and that he is receptive to the use of Windows-native alternatives, if it means the library can be ported.

# Details #

# Caveats #
  * Linux: root access rights are required to have libUSB connect to a Bluetooth modue. Running BTdaemon as root is not optimal, but probably ok for most people toying with BTstack.
  * OS X: The OS sizes the BT module when plugged in.
    * For OS 10.7+ and higher, please follow the instructions here: http://developer.apple.com/library/ios/#technotes/tn2295/_index.html
    * For older systems, the best choice is to unload the Bluetooth kernel module /System/Library/IOBluetoothFamily.kext using kextunload

# Default Endpoints #
  * 0x00: Control Transfer: HCI Commands
  * 0x81: Interrupt Transfer: HCI Events
  * 0x02: Bulk Transfer: ACL packet to BT dongle
  * 0x82: Bulk Transfer: ACL packets from BT dongle

# Status #
  * Finding a BT module (either by product & vendor ID or by device class) works on Linux and Mac.
  * Probing the USB device descriptor for the endpoints of the the four HCI packet types works on Linux and Mac
  * Works with current version of libusb and libusbx in both polling as well as fd selects.
  * ISSUE: It's not possible to send HCI packets larger than 83 bytes and I don't know why (or if it's in libusb or in BTstack code). Receiving larger packets isnt' a problem.

# TODO #
  * More tests
  * Fix issue of sending packets larger than 83 bytes
  * more documentation
  * libusb is supported on win32 - have someone try that out