# Introduction #

## <font color='red'><b>NEW!</b> Get the documentation for embedded systems: <a href='http://bluekitchen-gmbh.com/docs/btstack-gettingstarted-1.0.pdf'>BTstack Manual v1.0</a>.</font> ##

**BTstack** is a Bluetooth stack that is mainly designed for embedded devices where resources are scarce. Despite scarce resources, it allows multiple applications to make use of Bluetooth services at the same time. The BTstack architecture consists of a Bluetooth server, called **BTdaemon**, which handles client requests, and a corresponding client library, which forwards the requests to the BTdaemon. In the following, the design of the BTdaemon is explained in more detail.


# BTdaemon Architecture #


The **BTdaemon** is necessary to handle multiple clients. Clients use the BTstack client library to communicate with the BTdaemon over TCP or Unix sockets.
In addition to necessary **socket server**, the other main components of the BTdaemon are: a **run loop** and the actual **Bluetooth stack**. The Bluetooth stack and the socket server are implemented as a collection of finite state machines. The state machines are executed by a single run loop, where all sockets and the Bluetooth module are registered as individual data sources. This design allows for a single-threaded implementation, which avoids concurrency problems and facilitates porting and running BTstack on embedded systems. As embedded systems usually implement a single functionality, support for multiple clients results in an unnecessary overhead. In this case, e.g., to support 8-bit microcontrollers, the socket server can be omitted and the client directly integrated into the BTdaemon run loop.

![http://btstack.googlecode.com/svn/files/BTstack_architecture.png](http://btstack.googlecode.com/svn/files/BTstack_architecture.png)

## Posix Run Loop Implementation ##

In the widely spread Posix standard, everything (files, I/Os, sockets, devices) is accessed via file descriptors. The BTdaemon's run loop is exploiting this fact to manage multiple data sources without multithreading. Such a data source is defined by a file descriptor and an associated data\_source\_ready handler:

```
typedef struct data_source {
    linked_item_t item;                                 // <-- used internally by RunLoop
    int  fd;                                            // <-- file descriptors to watch or 0
    int  (*data_source_ready)(struct data_source *ds);  // <-- handler
} data_source_t;
```


All data sources are tracked in parallel using the [select()](http://www.manpagez.com/man/2/select/) call. For a detailed introduction into handling of multiple file descriptors within a single thread, check this [single-threaded socket server](http://www.lowtek.com/sockets/select.html).

Besides data sources, the BTdaemon needs to track various Bluetooth-related timeout events. For example, the BTdaemon supports an option to turn off the Bluetooth module one minute after the last client disconnects. The tracking of timeouts is also performed by the run loop.

For this, the run loop maintains a list of timeout events associated with the timeout (callback) handler and an optional callback argument. The list is ordered by time, such that the earliest timeout appears as the first element. The run loop then uses the interval between the current time and the earliest timeout as timeout parameter in the select() call. After select() returns, the run loop calls the callback handlers of all file descriptors returned by the select() are called, and those of the expired timeouts.

The described run loop approach is compatible with many GUI frameworks, notably Cocoa on Mac OS X and CocoaTouch on the iPhone. As the BTstack client library is also designed to be executed by a run loop, it can be readily integrated into GUI applications.

### Flow Control ###
Handling flow control for various data streams in a Bluetooth stack is tedious job. So far, BTstack implements:
  * Flow control for ACL and Command packets in HCI
  * Flow control for L2CAP data
  * Flow control for RFCOMM data
  * Flow control for Socket Server: As the socket server can receive messages from the clients at any time, a command or a data packet might be received while the Bluetooth module is already busy. If the socket server would simply skip reading data this time, it would be called again immediately by the run loop, as the socket connection is still ready. Therefor, the BTdaemon has to _block_ this socket connection by removing its data source from the run loop. As soon as the Bluetooth module is ready again, the data source can be added back to the run loop.

## Shared state ##

The state shared between all connected clients consists of:
  * List of Baseband connections, and their state: active/sniffing/hold/park
  * Bluetooth state: off/initializing/on/terminating
  * Configuration: page scan timeout, discoverable, etc.
  * List of L2CAP channels in listening mode (a.k.a server)
  * List of active L2CAP channels
  * List of active RFCOMM channels (not implemented yet)


## Modular Design ##

The BTstack was designed in a modular fashion: it provides an interface for "pluging-in" different run loop, HCI transport, and Bluetooth hardware control implementations. The run loop has to be set first by calling run\_loop\_init({RUN\_LOOP\_POSIX, RUN\_LOOP\_COCOA}). The HCI layer is constructed with a pointer to an hci\_transport\_t structure, a pointer to an HCI transport configuration structure and a pointer to an bt\_control\_t structure.


### HCI Transport API ###

The HCI Transport API allows the BTdaemon to use different transport implementations. The hci\_tranport\_t control structure contains a data\_source\_t structure used for the single-thread implementation.

```
typedef struct {
    int    (*open)(void *transport_config);
    int    (*close)();
    int    (*send_packet)(uint8_t packet_type, uint8_t *packet, int size);
    void   (*register_packet_handler)(void (*handler)(uint8_t packet_type, uint8_t *packet, int size));
    const char * (*get_transport_name)();
} hci_transport_t;
```

Note: packet sizes are not needed for HCI packets (ACL, Command, Event), as they themselves contain length information. They could be removed from the function parameters, but are kept for efficient code.

### Bluetooth Hardware Control API ###

The Bluetooth hardware control API allows the HCI layer to initialize, turn on/off the Bluetooth module, and configure a particular transport protocol including the physical connection (H4/H5/USB, device path, baud rate, flow control, Vendor ID, Product ID). This is device specific, e.g., turning on Bluetooth on the iPhone requires to make use of Apple's BlueTool; on the BTnode, the power is controlled by a special kernel function, etc.


## Goodies ##

### HCI Logging ###

**BTdaemon** is able to write HCI dumps in several formats. It supports both the bluez-hcidump format, which can be read by: the hcidump tool, the GNOME Bluetooth-Analyzer, the Wireshark, and the Apple's PacketLogger format. In addition, it can dump raw HCI packets in hex.

### Development/Debugging ###

So far, only Bluetooth modules connected by a serial connection (H4/H5) are supported. To communicate with USB Bluetooth modules, which are not supported yet, the cross-platform libUSB library can be employed.

An early version of an HCI USB Transport implementation is included and should simplify the development/debugging of BTstack applications on standard desktops. More on it can be found on the [HCI USB Transport page](HCI_USB_Transport.md).


## References ##

  * [Official Bluetooth specification](https://www.bluetooth.org/Technical/Specifications/adopted.htm)
  * CSR provides a wealth of documents on HCI and their hardware after registration (free account) on their [CSR Support Site](http://www.csrsupport.com).
  * [BlueZ - The GPL Linux Bluetooth Stack](http://www.bluez.org/)
    * ~~The [BlueZ Wiki](http://wiki.bluez.org/wiki)~~ seems to be gone (July 2011)
    * The implementation for CSR lists some of the magical properties set by the Apple BlueTool init scripts, I could not find in the CSR documentation.
  * The lightweight Bluetooth Stack (lwBT). An embedded BT stack, intended for use with the lightweight IP stack (lwIP). Not developed anymore, website gone. It is still used, e.g., in the [dynawa open-source watch](http://dynawa.org) -[repository](http://code.google.com/p/dynawa). Also, it is/was used by all iPhone Bluetooth projects before BTstack.
  * The ETH BTnut BT Stack: http://www.btnode.ethz.ch
    * Documentation on the HCI implementation: http://www.btnode.ethz.ch/static_docs/btnut/bt-stack.pdf
    * [HCI API](http://www.btnode.ethz.ch/static_docs/doxygen/btnut/bt__hci__api_8h.html)
    * [HCI Commands](http://www.btnode.ethz.ch/static_docs/doxygen/btnut/bt__hci__cmds_8h.html/group__btnode__resources.html#btnode_bt-stack)
  * [libUSB](http://libusb.wiki.sourceforge.net/)