# Introduction #

L2CAP is primarily concerned with:
  1. Protocol multiplexing
  1. Segmentation and reassembly operations

We do not intend to support per-channel flow control or retransmissions. Therefore we will only implement L2CAP Basic Mode. The L2CAP State machine will follow the overall goal of non-blocking operations.

## Client API ##
All L2CAP commands are forwarded to the BTdaemon with the bt\_send\_cmd. Maybe explicit function calls will be provided at a later time - using a code generator.

Have a look at the example code: http://code.google.com/p/btstack/source/browse/trunk/example/l2cap-server.c

### Outgoing Connections ###
To open an L2CAP connection use the l2cap\_create\_channel command:
```
  // create channel
  bt_send_cmd(&l2cap_create_channel, bd_addr, uint16 psm);

  // transfer
  l2cap_send(uint16_t source_cid, uint8_t *data, uint16_t len);
```

On success, an HCI\_EVENT\_L2CAP\_CHANNEL\_OPENED event will generated which contains the locall\_cid. Data received on the channel will be received by the data callback. To send data over the channel, use the l2cap\_send function.

### Incoming Connections ###
To register an L2CAP service, send the l2cap\_register\_service(uint16\_t psm, uint16\_t mtu) command. The incoming MTU is ignored for now. On incoming L2CAP connection request, an L2CAP\_EVENT\_INCOMING\_CONNECTION event will be received by the application event handler. The application is then free to accept or decline the request using the l2cap\_decline/accept\_connection commands.

```
  bt_send_cmd(&l2cap_register_psm, uint16_t psm, uint16_t incoming_mtu);
  bt_send_cmd(&l2cap_unregister_psm, uint16_t psm);

  bt_send_cmd(&l2cap_accept_connection, uint16_t source_cid);
  bt_send_cmd(&l2cap_decline_connection, utin16_t source_cid, uint8_t reason);
```

### Events ###
L2CAP emits the following events:
```
#define L2CAP_EVENT_CHANNEL_OPENED                         0x70
// data: event (8), len(8), status (8), address(48), handle (16), psm (16), local_cid(16), remote_cid (16) 

#define L2CAP_EVENT_CHANNEL_CLOSED                         0x71
// data: event (8), len(8), channel (16)

#define L2CAP_EVENT_INCOMING_CONNECTION			   0x72
// data: event (8), len(8), status (8), address(48), handle (16), psm (16), local_cid(16), remote_cid (16) 

#define L2CAP_EVENT_TIMEOUT_CHECK                          0x73
// data: event(8), len(8), handle(16)

#define L2CAP_EVENT_CREDITS				   0x74
// data: event(8), len(8), local_cid(16), credits(8)
```

# Details #

## L2CAP State Machine ##
Most part of the L2CAP State Machines will be implemented as part of the BTdaemon. For each connection (channel), a separate state machine is required. On the client side, a subset of state information is required, too.

### Dispatching L2CAP Signaling Commands to Channels State Machines ###
To process incoming L2CAP Signaling Commands, they need to be dispatched to the responsible Channel state machine. The following lists the different commands and how they are dispatched:
  1. command reject: id
  1. connection request: PSM - not a particular channel
  1. connection response: id
  1. configuration request: dest\_cid == src\_cid
  1. configuration response: id
  1. disconnect request: dest\_cid == src\_cid
  1. disconnection response: id
  1. echo request: Stack        - not a particular channel
  1. echo response: id          - not a particular channel
  1. information request: Stack - not a particular channel
  1. information response: id   - not a particular channel

Commands 0x08-0x0b are not channel related, also 0x02 is used to setup a new channel. We only have to react to {0x01-0x07}\{0x02}. For command codes with odd number, we match by the signal identifier sent earlier by the channel - the channel\_t struct stores the last sent signal identifier. Otherwise, we match by the destination channel ID.

## Credit-based Flow Control ##

Although HCI will prevent you from overrunning the Bluetooth controller, it's more efficient to use the credit-based flow control provided by the BTstack L2CAP layer.

As there's no benefit of having more than 3 packets buffered in the Bluetooth module ready to be send, L2CAP emits single packet credits (L2CAP\_EVENT\_CREDITS)to the application which then is allowed to send a single packet. The first credit is emitted after the channel is opened.

## Packet Buffers at BTdaemon ##

Theory:
  * One ACL buffer for incoming packets from Bluetooth chipset (in hci\_transport\_h4.c)
  * For each baseband connection:
    * One L2CAP packet buffer of size max MTU(X) (X is set of all L2CAP connections on this baseband connection). It is needed for assembly of received ACL packets into an L2CAP packet. (hci\_connection\_t in hci.h)
  * For each client connection: one L2CAP packet buffer of size MTU (struct connection in socket\_connection.c)

Practice:
  * BTstack only supports L2CAP packets with max MTU <= 339 bytes(= DH5 packet)
  * Update: the SVN version uses 1021 bytes packets (= 3-DH5 packets)

Possible optimizations:
  * Single ACL buffer in HCI transport unnecessary when access to L2CAP buffer possible

## References ##
  * Official Bluetooth specification and other open stacks, see [Architecture](Architecture.md)
  * Nice [L2CAP Summary](http://www.inf.ethz.ch/personal/kasten/research/bathtub/bluetooth/)