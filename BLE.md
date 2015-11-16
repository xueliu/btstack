# BTstack and Bluetooth 4.0 - Bluetooth Smart Devices #
## <font color='red'><b>NEW!</b> Get the documentation for embedded systems: <a href='http://bluekitchen-gmbh.com/docs/btstack-gettingstarted-1.0.pdf'>BTstack Manual v1.0</a>.</font> ##

BTstack provides early support for Bluetooth Smart devices, both as single mode or dual mode stack.

Bluetooth Smart devices implement one or more GATT profiles. These GATT profiles define a set of one or more Services. Each Service in turn defines a set of Characteristics. These profiles have a similar role as the SDP records in classic Bluetooth.

In BTstack, a GATT profile compiler converts a textual representation of the GATT Services and Characteristics into a compact internal database used by the ATT protocol server.

The ATT protocol server answers incoming ATT requests based on information provided in the compiled database and provides read- and write-callbacks for dynamic attributes.

## GATT Profiles ##
GATT profiles are defined by a simple textual comma separated value (.csv) representation. While the description is easy to read and edit, it is compact and can be placed in ROM.

The current format is:
```
PRIMARY_SERVICE, {SERVICE_UUID}
CHARACTERISTIC, {ATTRIBUTE_TYPE_UUID}, {PROPERTIES}, {VALUE}
CHARACTERISTIC, {ATTRIBUTE_TYPE_UUID}, {PROPERTIES}, {VALUE}
...
PRIMARY_SERVICE, {SERVICE_UUID}
CHARACTERISTIC, {ATTRIBUTE_TYPE_UUID}, {PROPERTIES}, {VALUE}
...
```


Properties can be a list of READ | WRITE | WRITE\_WITHOUT\_RESPONSE | NOTIFY | INDICATE | DYNAMIC

Value can either be a string ("this is a string"), or, a sequence of hex bytes (e.g. 01 02 03)

UUIDs are either 16 bit (1800) or 128 bit (00001234-0000-1000-8000-00805F9B34FB)

Example for a [Device Information Service](https://developer.bluetooth.org/gatt/services/Pages/ServiceViewer.aspx?u=org.bluetooth.service.device_information.xml)

```
PRIMARY_SERVICE, 180A
CHARACTERISTIC, 2A29, READ, "Manufacturer Name String"
CHARACTERISTIC, 2A24, READ, "Model Number"
CHARACTERISTIC, 2A25, READ, "Serial Number"

// more strings

// PnP ID, see https://developer.bluetooth.org/gatt/characteristics/Pages/CharacteristicViewer.aspx?u=org.bluetooth.characteristic.pnp_id.xml
// Vendor ID Source = 1, Vendor ID = 0x1234, Product ID = 0x5678, Product Version = 0x0001
CHARACTERISTIC, 2A50, READ, 02 34 12 78 56 01 00

```

Reads/writes to a Characteristic that is defined with the DYNAMIC flag, are forwarded to the application via callback. Otherwise, the Characteristics cannot be written and return the specified constant value.

Adding NOTIFY and/or INDICATE automatically create an addition Client Configuration Characteristic.

See `profile.gatt` in the example folder and the GATT compiler `compile-gatt.py`.

## Mapping of Characteristics to Handles ##
BTstack only provides an ATT Server, while the GATT Server logic is mainly provided by the GATT compiler. While GATT identifies Characteristics by UUIDs, ATT uses Handles (16 bit values). The GATT compiler creates a list of defines in the generated **.h file, so you don't need to hard-code the attribute handles.**

## ATT Protocol ##
ATT sits on top of L2CAP. For a single-mode Bluetooth 4.0 device, most of L2CAP functionality is not used. To save on ROM space, a limited version of L2CAP is provided as l2cap\_le.c in the ble folder.

## Example ##
`MSP-EXP430F5438-CC256x/example-ble` provides a ready-to-run example for a test Peripheral device. It assumes that a PAN1323 module with a CC2564 chipset is plugged into the RF socket.

The profile of this device defines 3 dynamic Characteristics. A write to the characteristic with the UUID fff1 (handle 0x000b) sets the displayed text on the LCD display, a write to the characteristics fff2 (handle 0x000d) controls an LED on the board. The third Characteristic only shows how to specify a 128-bit UUID.

[Video of the example](http://www.youtube.com/watch?v=xWY5GinDCQc)

## Next Steps ##
Currently, the complete Low Energy configuration of the Bluetooth module is handled by the `ble_server` example application. Most of this should be moved into `hci.c`. It isn't clear how much of  those LE setup commands need to be customized in real applications.