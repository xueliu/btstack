# Introduction #

## <font color='red'><b>NEW!</b> Get the documentation for embedded systems: <a href='http://bluekitchen-gmbh.com/docs/btstack-gettingstarted-1.0.pdf'>BTstack Manual v1.0</a>.</font> ##

BTstack was developed and primarily used on OSs with POSIX file I/O (iOS had almost 2 million downloads recently). Smaller embedded systems luck such luxury. When running on a device without any OS some things have to adapt. Examples for such a system might be the Arduino platform, if a bigger MCU like the ATmega 2561 is used, or the [BTnode](http://btnode.ethz.ch), or, your favorite ARM-developer board. :)

# Prototypes #
First run of BTstack on embedded systems without OS:
  * [Stellaris Cortex M3](http://www.youtube.com/watch?v=NlOcoKWuZhU)
  * [TI-MSP430F5438](http://www.youtube.com/watch?v=j7mJuklrIxw)

# Details #
The embedded port is in an early stage, but fully functional.

For use without an OS and a Bluetooth module connected via an UART, you need to implement:
  * hal\_tick.h
  * hal\_cpu.h
  * hal\_uart\_dma.h

## Run Loop ##
BTstack uses a run loop to coordinate the handing of incoming data and to schedule work.

The run loop handles events from two different types of sources: data sources and timers.
Data sources represent communication interfaces like an UART or an USB driver. Timers can be used to handle periodic events and are use by BTstack to implement various Bluetooth-related timeouts. For example, a Bluetooth base- band channel without an active L2CAP channel is disconnected after 20 seconds. Data sources and timers are represented by the two structs data source t and timer source t respectively. Each of these structs have a link list node and a pointer to a callback function. All active timers and data sources are kept in a link list. While the list of data sources is unsorted, the timers are are sorted by timeout for efficient processing.

The complete run loop cycle looks like this: first, the process function of all registered data sources are called in a round robin way. Then, the callback functions of timers that are ready are executed. Finally, it will be checked if another run loop iteration has been requested in a critical section block. If no addition data source became ready, the run loop will enter a sleep mode.

Timers are also single shot: timers will be removed from the timer list before the event handler callback is called. If you need a repeating timer, you can re-register the same timersource in the callback function again. Note that BTstack expects to get called periodically to keep its time (see for more HW abstraction layer).

Incoming data over the UART/USB or timer ticks will generate an interrupt and wake up the micro controller. In order to avoid the situation where a data source becomes ready just before the run loop enters sleep mode, an interrupt-driven data source has to call\_embedded\_trigger(). The call to embedded trigger sets an internal flag that is checked in the critical section just before entering sleep mode.

To enable the use of timers, make sure that you defined HAVE\_TICK in the config file.

```
void run_loop_set_timer_handler(timer_source_t ∗ts, void (∗process)( timer_source_t ∗ ts));
￼￼￼￼￼￼
void run_loop_set_timer(timer_source_t ∗ts, uint32_t timeout_in_ms); 
void run_loop_add_timer(timer_source_t ∗ts);
void run_￼loop_set_data_source_handler(data_source_t ∗ds , int (∗ process)(data_source_t ∗ ds));
void run_loop_add_data_source(data_source_t ∗ds);
```

You’ll need only to start the run loop explicitly in your startup code after the initialization.
The application can as well register data sourced and timers, e.g. periodical sampling of sensors, or communication over the UART.

### Tick Hardware Abstraction Layer ###
BTstack requires a way to learn about passing time. In an embedded configuration, the following functions have to be provided. The hal\_tick\_init() and the hal\_tick\_set\_handler() will be called during the initialization of the run loop.

```
void hal_tick_init(void);
void hal_tick_set_handler(void (*tick_handler)(void));
int  hal_tick_get_tick_period_in_ms(void);
```

## Memory Configuration ##

BTstack has to keep track of services and active connections on the various protocol levels. In addition, the non-persistent database for remote device names and link keys needs memory. The structs for each type of data can be allocated in two different manners:
  * statically from an individual memory pools, whose maximal number of elements is defined in config.h
  * dynamically using the malloc/free functions, if HAVE\_MALLOC is defined in config.h

If both HAVE MALLOC and maximal size of a pool are defined in the config file, the statical allocation will take precedence. In the case that both are omitted, an error will be raised.
To initialize the static pools you need to call btstack\_memory\_init() function.
An example of a memory configuration in config.h file for a single SPP service looks like this:

```
#define HCI_ACL_PAYLOAD_SIZE 52
#define MAX_SPP_CONNECTIONS 1
#define MAX_NO_HCI_CONNECTIONS MAX_SPP_CONNECTIONS
#define MAX_NO_L2CAP_SERVICES  2
#define MAX_NO_L2CAP_CHANNELS  (1+MAX_SPP_CONNECTIONS)
#define MAX_NO_RFCOMM_MULTIPLEXERS MAX_SPP_CONNECTIONS
#define MAX_NO_RFCOMM_SERVICES 1
#define MAX_NO_RFCOMM_CHANNELS MAX_SPP_CONNECTIONS
#define MAX_NO_DB_MEM_DEVICE_NAMES  0
#define MAX_NO_DB_MEM_LINK_KEYS  3
#define MAX_NO_DB_MEM_SERVICES 1
```

## UART Hardware Abstraction Layer ##

On embedded systems, a Bluetooth module can be connected via USB or an UART port. BTstack implements two UART based protocols for carrying HCI commands, events and data between a host and a Bluetooth module: HCI UART Transport Layer (H4) and H4 with eHCILL support, a lightweight low power variant by Texas Instruments.

### HCI UART Transport Layer (H4) ###
Most embedded UART interfaces oper- ate on the byte level and generate a processor interrupt when a byte was received. In the interrupt handler, common UART drivers then place the received data in a ring buffer and set a flag for further processing or notify the higher-level code, i.e. in our case the Bluetooth stack.

Bluetooth communication is packet-based and a single packet may contain up to 1021 bytes. Calling a data received handler of the Bluetooth stack for every byte creates an unnecessary overhead. To avoid that, a Bluetooth packet can be read as multiple blocks where the amount of bytes to read is known in advance. Even better would be the use of on-chip DMA modules for these block reads, if available.

The BTstack UART HAL API reflects this design approach and the underlying UART driver has to implement the following API:

```
void hal_uart_dma_￼init(void) ;
void hal_uart_dma_set_block_received(void (∗block_handler)(void));
void hal_uart_dma_set_block_sent(void (∗block_handler)(void));
int hal_uart_dma_set_baud(uint32_t baud) ;
void hal_uart_dma_send_block(const uint8_t ∗buffer , uint16_t length); 
void hal_uart_dma_receive_block(uint8_t ∗buffer, uint16_t len);
```

The main HCI H4 implementations for embedded system is hci\_h4\_transport\_dma(). This function calls the following sequence: hal\_uart\_dma\_init(), hal\_uart\_dma\_set\_block\_received() and hal\_uart\_dma\_set\_block\_sent(). After this sequence, the HCI layer will start packet processing by calling hal\_uart\_dma\_receive\_block(). The HAL implementation is responsible for reading the requested amount of bytes, stopping incoming data via the RTS line when the the requested amount of data was received and has to call the handler. By this, the HAL implementation can stay generic, while requiring only three callbacks per HCI packet.

### H4 with eHCILL support ###

With the standard H4 protocol interface it is not possible for either the host nor the baseband controller to enter the sleep mode. Besides the official H5 protocol various chip vendors came up with proprietary solutions to this. The eHCILL support by Texas Instruments allows both the host and the baseband controller to independently enter sleep mode without loosing their synchronization with the HCI H4 Transport Layer. In addition to the IRQ-driven block-wise RX and TX, the eHCILL requires a callback for CTS interrupts.

```
void hal_uart_dma_set_cts_irq_handler(void (∗cts_irq_handler)(void));
void hal_uart_dma_set_sleep(uint8_t sleep );
```


## Arduino ##
The Arduino firmware provides interrupt-driven UART drivers with an 128 byte read buffer. It does not support CTS/RTS. Writes are blocking. A better driver is needed but possible.