# Introduction #

The service discovery protocol is used to learn about services provided by remote devices and is required whenever a BTstack-based app wants to implement a standard Profile like HID or RFCOMM for other devices.

# Details #
It basically consists of a simple request-response scheme with the option of segmenting a response into multiple packets. A fully implemented but untested SDP server was added with [r709](https://code.google.com/p/btstack/source/detail?r=709).

# Limitations #
The SDP Server does not provide a service record for itself. Let me know, if you are missing this feature for a particular use case.

# Usage #
To register a service record, you first need to create one. For this, you can either use a hard-coded representation or use the data element (de) functions in sdp\_utils.h

The functions in sdp\_utils.c have been designed with the goal of minimal memory footprint and allow to create a service record in-place without using additional structures. However, they are not exactly user-friendly.

In OS X, a service record can be described by Property Lists (plist), which can be read into an NSDictionary. By this, the process of creating a service record is made easier as Apple's PropertyList editor can be used. For the Cocoa(Touch) platform, this can be implemented. If someone does it before me, please tell me.