## <font color='red'><b>NEW!</b> Get the documentation for embedded systems: <a href='http://bluekitchen-gmbh.com/docs/btstack-gettingstarted-1.0.pdf'>BTstack Manual v1.0</a>.</font> ##

# Hardware Setup #
We assume that a PAN1315, PAN1317, or PAN1323 is plugged into the provided port and the "RF3 Adapter board" is used or at least simulated. See http://processors.wiki.ti.com/index.php/PAN1315EMK_User_Guide#RF3_Connector

# General Tools #

The MSP430 port of BTstack is developed using the Long Term Support version of of mspgcc. General information about it and installation instructions are provided on the [MSPGCC Wiki](http://sourceforge.net/apps/mediawiki/mspgcc/index.php?title=MSPGCC_Wiki)

On Unix-based systems, Subversion, make, and Python are usually installed. If not, use the system's packet manager to install them.

On Windows, you need to install and configure Subversion, make, and Python manually:
  * Subversion for Windows: http://www.sliksvn.com/en/download
  * Optionally Tortoise SVN: This is a GUI frontend for Subversion that makes checkouts and other operations easier by integrating them into the Windows Explorer: http://tortoisesvn.net/downloads.html
  * GNU Make for Windows:
    * Download from http://gnuwin32.sourceforge.net/packages/make.htm
    * Add its bin folder to the Windows Path in Environment variable. The bin folder is where make.exe resides, and it's usually located in "C:\Program Files\GnuWin32\bin". Add this path to the Path environment variable as described later.
  * Python for Windows:
    * Download from http://www.python.org/getit/
    * Add python installation folder to the Windows Path in Environment variable. For example, for one python installation the path is "C:\Python27". Add this path to the Path environment variable as described later
  * mspgcc:
    * Download MSPgcc for Windows from here:http://sourceforge.net/projects/mspgcc/files/Windows/mingw32/
    * Extract to c:\mspgcc
    * Add C:\mspgcc\bin folder to the Windows Path in Environment variable.
  * Adding paths to the Windows Path variable:
    * Go to Control Panel -> System -> Advanced tab -> Environment Variables.
    * In the top part you will see a list of User variables
    * Click on the Path variable and then click edit.
    * Go to the end of the line, then append "C:\mspgcc\bin" to the list without the quotes.
    * Ensure that there is a semicolon before and after "C:\mspgcc\bin".

# Getting BTstack from SVN #

Use Subversion to get a check out of the latest version. There are two approaches:
  1. Use subversion in a command line: Simply go to a folder where you would like to download BTstack, then type: `svn checkout http://btstack.googlecode.com/svn/trunk/`
  1. Windows-only: Use tortoise svn (to checkout by using the URL http://btstack.googlecode.com/svn/trunk/

In both cases, subversion will create a folder and pull all the code there.

# CC256x Init Scripts #

In order to use the CC256x device inside of the PAN13xx modules, an initialization script must be obtained. Due to licensing restrictions, this initialization script must be obtained as follows:
  * Download the appropriate BTS file from: http://processors.wiki.ti.com/index.php/CC256x_Downloads
  * Copy the included .bts file into chipset-cc256x
  * Run the provided Python script: ./convert\_bts\_init\_scripts.py that's in chipset-cc256x
  * Add bt\_control\_cc256x.c plus bluetooth\_init\_cc2560\_2.44.c (or newer) or bluetooth\_init\_cc2560A\_2.1.c (or newer) to your project. If you have a PAN1315, use cc2560.
  * Use bt\_control\_cc256x\_instance() to get a bt\_control\_t instance and use it in hci\_init() call

# Compiling the Examples #

Open a command prompt in the MSP-EXP430F5438-CC256x/example folder and run make. If all the paths are correct, it will generate several hex files. These files are binary and can be downloaded to the MSP430 device, as per the next section.<br>

<h1>Downloading Binary</h1>

To download binary files to the MSP430 MCU, you need a programming tool, the MSP430 USB&nbsp;FET debugger or other programmer connected, and the MSP430F5438 Experimenter board or other board powered to do so.<br>
<br>
<ul><li>MSP430Flasher (windows-only):<br>
<ul><li>Download the binary version of the MSP430Flasher software: <a href='http://processors.wiki.ti.com/index.php/MSP430_Flasher_-_Command_Line_Programmer'>http://processors.wiki.ti.com/index.php/MSP430_Flasher_-_Command_Line_Programmer</a><br>
</li><li>Use the command <code> MSP430Flasher.exe -n MSP430F5438A -w "INSERT_NAME_OF_BINARY_OUT_FILE_HERE.hex" -v -g -z [VCC] </code>
</li><li>Modify the name with the name of your application and execute the binary file.<br>
</li></ul></li><li>MSPdebug:<br>
<ul><li><a href='http://mspdebug.sourceforge.net/'>http://mspdebug.sourceforge.net/</a>
</li><li>Example session with the MSP-FET430UIF connected on OS X:<br>
<pre><code>mspdebug -j -d /dev/tty.FET430UIFfd130 uif<br>
... <br>
prog blink.hex<br>
run<br>
</code></pre></li></ul></li></ul>

<h1>List of Examples</h1>
<ul><li>led_counter: no Bluetooth, just a test to see if the timer interrupt and the debug UART work<br>
</li><li>hid_demo: on start, the device does a device discovery and connects to the first Bluetooth keyboard it finds, paris, and allows to type on the little LCD screen.<br>
</li><li>spp_counter: provides a virtual serial port via SPP. On connect, it sends a message every second<br>
</li><li>spp_accel: provides a virtual serial port via SPP. On connect, it sends the current accelerometer values as fast as possible<br>
</li><li>spp_flowcontrol: provide a virtual serial port via SPP. On connect, it echoes all received data on the debug UART but adds a 1/4 pause after each packet. This was used to test that flowcontrol works.</li></ul>

<h1>Next Steps</h1>
You can learn more about BTstack by reading the other wiki pages. We recommend the <a href='Architecture.md'>Architecture</a> overview next and the details about using the packet handler to execute a sequence of command on GettingStarted.