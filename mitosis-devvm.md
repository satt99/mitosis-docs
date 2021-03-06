# Creating a VM for Mitosis Development

## The Goal

The goal of this document is to allow keyboard enthusiasts to create a well
defined Linux Virtual Machine that will allow for the smooth development of
keyboard software for the Mitosis. This will allow you use a well-defined, 
proven working Linux system, on your computer.

## Document Status

This document is not yet complete, untested, and needs some work. 
I believe the parts from installing VirtualBox up to but not including
flashing the wireless modules is fairly complete and working as written.

I don't yet have a wireless module to test flashing, so that part is 
unfinished and untested. It will be updated as I try it out and learn more.

This document may be useful or misleading, I am hoping it is useful.

TODO: Start over creating a new VM from scratch, to test the doc.

TODO: Check the doc organization.

## Definitions
  * Mitosis - A 2-piece wireless, programable (QMK), keyboard that communicates
to a wireless "dongle" plugged into a USB port.
  * VM - Virtual machine
  * Host system (or simply host) - 
  * Wireless module - 
  * ST-LINK V2 - an in-circuit debugger/programmer
  * TODO: add more here...

## Install VirtualBox, make a new VM, install Ubuntu:

### Download and Install VirtualBox

* Download the app for your host system here: https://www.virtualbox.org/wiki/Downloads
* Version 5.1.26
* (I'm using the macOS version, any there should work the same. Use the one that will run on your computer.)
* download
* open (double-click) the .dmg (macOS) or other file
* install (double-click) the VirtualBox.pkg file, verifies the package. It will
take a little while to start up.
* the installer runs...I selected "Install for all users...; the default
installation

### Run and configure VirtualBox

TODO: Check the order / values of these steps.

* Run VirtualBox (In Applications folder on macOS, or other)
* Click "New" near top-left of window.
* Name your VM as you like. I'll be using "MitosisDev"
* Select "Type: Linux"
* Select "Version: Ubuntu (64-bit)", Continue
* Select RAM to allocate for the running VM. I'll start with 1542 MB, Continue
* Select "Create a virtual hard disk now", Create
* Selet "VHD" ("VDI" cannot be imported into other VMs later, if you wanted to.), Continue.
* Select "Dynamically allocated" (Only uses the disk space it needs.) Continue.
* Set value for the HD to 30GB, and the default name & location. Create.
* Network / Adapter 1: Enable, Attached to NAT
* Ports / Serial Ports: Disabled
* Ports / USB: Enable; USB 1.1
* Plug in your ST-LINK V2. Click the USB and "+" icon. Select the ST-LINK device to attach it to the VM (when running).
* In preferences, enable all tabs for the UI
```
     +-----+
     |photo|
     +-----+
```
* OK

### Within VirtualBox, create a VM
TODO: Check the order / values of these steps.

* VM Name: MitosisDev
* Disk name: mitosisdevhd
* User: mitosis
* Password: my easy os
* 2GB RAM
* 30GB HD, dynamic
* 64-bit OS
* ...

### Install Ubuntu v16.x into the VM

## Get a current Ubuntu installer ISO image and install Ubuntu

* Current version is: 16.04.3 LTS
* go to https://www.ubuntu.com/download/desktop and download the v16.x ISO file.
* Mount the ISO as a CD-ROM for your new VM
* Start up the VM, and boot from the Ubuntu ISO
* Install Ubuntu
* DO NOT mess with the Software & Updates system setting!
* **Use the System Settings to update Ubuntu to the latest software.** (TODO: more)
* Install guest additions, see below for instructions.
* Add Oracle VM VirtualBox Extension Pack, see below for instructions.

## Add the ST-LINK V2 programer to the Ubuntu VM as a USB device
* Launch VirtualBox
* Select your VM
* Select Settings
* Select Ports, and then USB
* Click the "Add" button on the right
* Select the ST-LINK USB device to add that hardware to the VM.
* **EVERYTIME** you open the VM, check the USB icon at the bottom of the window
or the Mac menu at the top of the screen to make sure the ST-LINK is attached.
(Checked) Or, check it in the VirtualBox settings for that VM under: Settings /
Ports / USB / USB Device Filters
    
-------------------------------------------

## Install Nordic SDK, and tool chain

The "tool chain" includes git, the gcc-arm compiler, and the OpenOCD programming
software that uses the ST-LINK V2 to flash the wireless modules.

Based off, but with added detail, reversebias README.md at: [https://github.com/reversebias/mitosis](https://github.com/reversebias/mitosis)

### Nordic SDK:

* Download **SDK v11** by going to:
[https://developer.nordicsemi.com/nRF5\_SDK/](https://developer.nordicsemi.com/
nRF5\_SDK/) (I found that the reversebias code does not compile with the current
v12 SDK. Looks like one of the .h files is in a different directory)
* **If** you want to try working with the **current** Nordic SDK use a browser
to go to [the Nordic
page](https://www.nordicsemi.com/eng/nordic/Products/nRF5-SDK/nRF5-SDK-v12-zip/
54291) selecting and downloading the latest SDK (12.3.0 August 2017). This ends
up in your ~/Downloads folder. You'll need to replace "11" with "12" in the
following instructions if you want to go that route. There will be other changes
you need to do to get it to work which are beyond the scope of this document.
Really, starting with v11 is simpler.
* go to the terminal and unzip the Nordic SDK from the Downloads folder into its
own directory by executing:
```
          cd ~
          unzip ~/Downloads/nRF5_SDK_11.0.0_89a8197.zip -d nRF5_SDK_11
          cd nRF5_SDK_11
```

### Install the required tool chain utilities (git, OpenOCD, and gcc-arm):
  * Install git, OpenOCD, and the gcc-arm compiler by going to the terminal and executing:
```
          sudo apt-get update # make packages up to date
          sudo apt install git
          git --version # display git version
          sudo apt install openocd gcc-arm-none-eabi
```
Update the Nordic makefile (for Linux) to point to where the tools (gcc) live.
This is where apt installed them.

* Edit the Makefile by going to the terminal and executing:
```
          gedit ~/nRF5_SDK_11/components/toolchain/gcc/Makefile.posix
```
* Replace something like: 
```
          GNU_INSTALL_ROOT := /usr/local/gcc-arm-none-eabi-4_9-2015q1
```
with:
```
          GNU_INSTALL_ROOT := /usr/
```

-------------------------------------------

## Install the the reversebias Mitosis git repository code

Download (clone) the reversebias Mitosis git repository:

  * Clone the reversebias Mitosis git repository by going to the terminal and executing:
```
          cd ~/nRF5_SDK_11
          git clone https://github.com/reversebias/mitosis
```
  * Install the udev rules (copy from the mitosis git clone to the /etc directory):
```
          sudo cp mitosis/49-stlinkv2.rules /etc/udev/rules.d/
```
For this to compile, the directory ~/nRF5\_SDK\_11 needs to contain 
the sub-directory "mitosis" (that we just cloned from the github mitosis
project). So your directory should now look like this:
```
          .
          ..
          components/
          documentation/
          examples/
          external/
          license.txt
          mitosis/              # mitosis at this level.
          nRF5x_MDK_8_5_0_IAR.msi
          nRF5x_MDK_8_5_0_Keil4.msi
          svd/
```


-------------------------------------------

## Test the ST-LINK configuration and software

Plug in, or remove and re-plug in the programmer. (Its LED should light up!)

The programming header pins on the side of the keyboard, from top to bottom are:
```
            SWCLK
            SWDIO
            GND
            3.3V
```

It's best to remove the battery during long sessions of debugging, as 
charging non-rechargeable lithium batteries isn't recommended.

Launch a debugging session with:
```
			cd ~/nRF5_SDK_11
			openocd -f mitosis/nrf-stlinkv2.cfg
```
As openocd runs it should:

1. Cause the LED on the ST-LINK to illuminate

2. Give you output starting with what I'll call the "Standard start-up header":
	```
			Open On-Chip Debugger 0.9.0 (2015-09-02-10:42)
			Licensed under GNU GPL v2
			For bug reports, read
							http://openocd.org/doc/doxygen/bugs.html
			Info : The selected transport took over low-level target control. 
						 The results might differ compared to plain JTAG/SWD
			adapter speed: 1000 kHz
			Info : Unable to match requested speed 1000 kHz, using 950 kHz
			Info : Unable to match requested speed 1000 kHz, using 950 kHz
			Info : clock speed 950 kHz
	```

3. If all is going well, the output will end with something like this and
OpenOCD will continue to run:

```
            Info : nrf51.cpu: hardware has 4 breakpoints, 2 watchpoints
```
If that is not what you get, see the "ST-LINK Troubleshooting" section, below.

## Load the firmware into the Wireless module(s) -- Linux or Mac OS X 

TODO: Test for Linux too.

TODO: Test after I have hardware that will alllow openocd to keep running.

### In Terminal 1:

Navigate to cloned mitosis repo and run openocd.
```
          cd ~/nRF5_SDK_11
          openocd -f mitosis/nrf-stlink.cfg 
```
Expect something similar to the following response.

        Open On-Chip Debugger 0.10.0
        Licensed under GNU GPL v2
        For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
        Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
        adapter speed: 1000 kHz
        Info : Unable to match requested speed 1000 kHz, using 950 kHz
        Info : Unable to match requested speed 1000 kHz, using 950 kHz
        Info : clock speed 950 kHz
        Info : STLINK v2 JTAG v17 API v2 SWIM v4 VID 0x0483 PID 0x3748
        Info : using stlink api v2
        Info : Target voltage: 3.250331
        Info : nrf51.cpu: hardware has 4 breakpoints, 2 watchpoints


### In Terminal window 2:

```
        telnet localhost 4444
        Trying 127.0.0.1...
        Connected to localhost.
        Escape character is ']'.
        Open On-Chip Debugger
```

In telnet session (terminal window 2) run the following commands. Replace the
information in [square brackets] with your system and board-specific information:

        > reset halt
        target halted due to debug-request, current mode: Thread 
        xPSR: 0xc1000000 pc: 0x00012b98 msp: 0x20001c48
        > nrf51 mass_erase
        > flash write_image /Users/[username]/[path]/nRF5_SDK_11/mitosis/precompiled/precompiled-basic-[receiver, left, right].hex
        not enough working area available(requested 32)
        no working area available, falling back to slow memory writes
        wrote 16652 bytes from file /Users/[username]/[path]/nRF5_SDK_11/mitosis/precompiled/precompiled-basic-[receiver, eft, right].hex in 3.201579s (5.079 KiB/s)
        > reset
Flashing wireless units complete. Exit everything.

1. Terminal #2: Ctrl+reset"]" then "q" to quit
2. Terminal #1: Control-C to halt and quit

TODO: test the above, when hardware is available.

-------------------------------------------
When I tried with PhenixFire's wireless in VM

        > reset halt
        target state: halted
        target halted due to debug-request, current mode: Thread 
        xPSR: 0xc1000000 pc: 0x000010f4 msp: 0x20004000
        > nrf51 mass_erase
        Unknown device (HWID 0x0000008f)
        
        > flash write_image /home/mitosis/nRF5_SDK_11/mitosis/precompiled/precompiled-basic-receiver.hex
        not enough working area available(requested 32)
        no working area available, falling back to slow memory writes
        (programmer light flashes)
        wrote 16652 bytes from file /home/mitosis/nRF5_SDK_11/mitosis/precompiled/precompiled-basic-receiver.hex in 119.679726s (0.136 Kib/s)s (5.079 KiB/s)
        > reset

-------------------------------------------

## Development cycle for Wireless Module
### Fix the length, if needed.

First, check to see if the length defined in the keyboard file:
```
		~/nRF5_SDK_11/mitosis/mitosis-keyboard-basic/custom/armgcc/gzll_gcc_nrf51.ld 
```
has been fixed.

The line:
```
    	RAM (rwx) :  ORIGIN = 0x20000000, LENGTH = 0x8000
```
needs to be replaced with:
```
    	RAM (rwx) :  ORIGIN = 0x20000000, LENGTH = 0x4000
```
(0x4000 replaces 0x8000.)
 
If it still reads 0x8000 edit the file and change it You only have to do this once.

### How to select keyboard Left or Right

Edit the file: ~/nRF5_SDK_11/mitosis/mitosis-keyboard-basic/main.c

The top 2 lines read:
```
        #define COMPILE_RIGHT
        //#define COMPILE_LEFT
```
Uncomment one, comment out the other.

### Recompile and load.
 
      cd ~/nRF5_SDK_11/mitosis/mitosis-keyboard-basic
      ./program.sh
It will compile and start (trying) to download.


-------------------------------------------
## Manual programming

TODO: needed?

From the factory, these chips need to be erased. Makesure openocd is running in
a different terminal window, then:

          echo reset halt | telnet localhost 4444
          echo nrf51 mass_erase | telnet localhost 4444

From there, the precompiled binaries can be loaded:

          echo reset halt | telnet localhost 4444
          echo flash write_image `readlink -f precompiled-basic-left.hex` | telnet localhost 4444
          echo reset | telnet localhost 4444

Automatic make and programming scripts

To use the automatic build scripts:

          cd mitosis/mitosis-keyboard-basic
          ./program.sh

An **openocd** session should be running in another terminal, as this script sends commands to it.

-------------------------------------------

## Development cycle for QMK software

TODO: Clean-up and test. Orig. post in 2014. Make sure it is up to date. 

See: [Deskthority](https://deskthority.net/workshop-f7/how-to-use-a-pro-micro-as-a-cheap-controller-converter-like-soarer-s-t8448.html) for more info.

1. Install AVRdude for Linux
1. Clone QMK repository
2. Edit
3. Compile
4. Reset receiver (Pro Micro) to put in program mode
  * This takes hitting the reset button twice as it powers up. 
5. Download with avrdude pointing to your USB port
6. Test new keyboard layout.





-------------------------------------------

## Reducing VM disk space:

After all is done, reduce the VM memory to make sharing easier:

  * Save a snapshot of the VM... just in case
  * Google for: reducing ubuntu disk space
    * Remove Old Kernels (If No Longer Required)
    * Uninstall Apps & Games You Never Use (LibreOffce)
      * sudo apt-get remove package-name1 package-name2
      * sudo apt-get autoremove
    * Clean the APT Cache
      * sudo apt-get clean
    * Use A System Cleaner like BleachBit
      * Preview
      * Clean
    * See where all the space is being used: launch the Disk Usage Analyzer
      * apt-get install duc
      * duc index / -x -p # rebuild database
      * duc ui # show results, graph
      * duc ui /usr/bruce # etc
    * See the 10 largest files on the system with: find / -printf '%s %p\n'| sort -nr | head -10
    * localepurge for removing non-english
    * auto rotate, compress and delete your logs.

The volume can then be zipped. This reduces the size to about 40% of the original.

-------------------------------------------

## Guest additions

Copy/Paste between the hos OS and VM is not by default. You can enable it by installing them, then 
enabling copy/paste in the settings for your VM.

1. Launch VirtualBox, start your VM.
2. Mount the ISO image on the guest VM's CD drive by selecting Devices->Insert Guest Additions CD Image. 
This popped up a prompt in my VM to install them.
3.  Once you have installed Guest Additions, then you can enable copy/paste:
  * Start the VM
  * Go to Machine > Settings in the file menu.
  * Go to the General tab, then Advanced.
  * Set the Shared Clipboard setting to Bidirectional.

-------------------------------------------

## Install the extension pack 

Make sure it is the same version as your installed version of VirtualBox!

Oracle VM VirtualBox Extension Pack instructions available here: https://www.youtube.com/watch?v=mwKmxxRbvws

  * Download from: https://www.virtualbox.org/wiki/Downloads
  * Select: VirtualBox 5.1.26 Oracle VM VirtualBox Extension Pack
      * Download: All supported platforms
      * The file downloaded is, for example: Oracle_VM_VirtualBox_Extension_Pack-5.1.26-117224.vbox-extpack
  * Run VirtualBox
  * Make sure no VMs are running
  * Select Extensions "Tab"
  * To the right of the "Extensions Packages" list, use the "Adds new package" drop-down to add the Extension pack.
  * A standard open file dialog is presented, navigate to the downloads folder and select and open the downloaded (.vbox-extpack) file.
  * The dialog informs you the USB 2.0 and USB 3.0 and RDP (Remote desktop) wil be supported.
  * Click "Install." Scroll to the bottom of the license agreement, click "I Agree."
  * You may then have to enter your password to start the install.
  * Select and start your VM.

  * Install Oracle VM VirtualBox Extension pack for Remote Display to work. 

Start VM

-------------------------------------------

## ST-LINK Troubleshooting

If you try to start up OpenOCD, and it does not respond as expected, here is 
some troubleshooting information.

If the ST-LINK **has not been reset lately** (unplugged, re-plugged in) the
display will end with:
```
        <Standard start-up header>
        Error: open failed
        in procedure: 'init'
        in procedure: 'ocd_burner'
```
Just unplug it, and plug it back in, and try again.

If the board is **not plugged in** (or, plugged in wrong) you will instead receive something like:
```
        <Standard start-up header>
        Info : STLINK v2 JTAG v17 API v2 SWIM v4 VID 0x0483 PID 0x3748
        Info : using stlink api v2
        Info : Target voltage: 3.261457
        Error: init mode failed (unable to connect to the target)
        in procedure 'init' 
        in procedure 'ocd_bouncer'
```
Double-check the order of your ST-LINK to keyboard (or receiver) connections.

-------------------------------------------

## Updating the ST-LINK V2 firmware

TODO: more here.

Your ST-LINK V2 firmware could be down-rev. I'm not aware of this causing any
problems, but I upgraded mine anyway. I have seen JTAG v17 with SWIM v4 reported
as working from another builder. This was my initial configuration too.
I have two ST-LINK V2 devices. I left one
with the old firmware, and upgraded the other to the latest version. 
Both of my programmers were made in China, and have the pins in this order:
```
        +----+--+--+-----+
        | RST|1 | 2|SWDIO|
        | GND|3 | 4|GND  |
        |SWIM|5 | 6|SWCLK|
        |3.3V|7 | 8|3.3V |
        |5.0V|9 |10|5.0V |
        +----+--+--+-----+
```

Go to this URL to read about the latest firmware upgrade:
```
        http://www.st.com/en/embedded-software/stsw-link007.html
```
You will have to provide a valid email address so they can send you a confirmation message.
Download the .zip file.
Unzip it.

Follow the readme.txt instructions. 

### macOS
TODO: Add more. Retest.

* Open the AllPlatorms folder in the unziped folder.
* Right click and open the STLinkUpgrade.jar, a Java app.
* You'll get a dialog like this:

![Dialog box showing firmware update](images/ST-LINK_V2_update.png)

If the programs cannot see your ST-LINK, unplug it, and plug it back in. As soon as it's
plugged back in click "Open in update mode."

Select your ST-LINK from the drop-down list. (There is probably only one.)

Click "Upgrade"

Now the version numbers have now changed:
```
    Old:  Info : STLINK v2 JTAG v17 API v2 SWIM v4 VID 0x0483 PID 0x3748
    New:  Info : STLINK v2 JTAG v28 API v2 SWIM v7 VID 0x0483 PID 0x3748
                           ^^^^^^^^        ^^^^^^^
```

A good guide to flashing the Pro Micro can be found here : https://goo.gl/pquFWu it's for the let's split but the MiniDox is setup in the same way, so you can follow the instructions just fine. Just be sure to compile the MiniDox firmware rather than the let's split
