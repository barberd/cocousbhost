# CoCo USB Pak

## Description

This board is a cartridge for the Tandy Radio Shack TRS-80 Color Computer (CoCo), an 8 bit computer produced between 1980 and 1991. It provides an USB 2.0 Host controller, the CH376S, to the CoCo. This allows one to use USB devices with the Color Computer given software drivers for the device. See "Using the Board" below.

![Front View](images/cocousbpak.png?raw=true)

Schematic is available [here](kicad/cocousbpak.pdf).

Assembly routines for using the cartridge in your own software can be found in the [Software](Software) directory. This includes drivers for USB hubs, keyboards, mice, and mass storage devices (such as pendrives). Additionally, a software patch for the Color Computer 3's BASIC to use a USB keyboard for input can be found in this directory.

## Building the board

This is open source hardware, so its up to one to get the board fabricated and then solder the components onto it. All components are through-hole-mount for easy assembly by hobbyists, with the exception of the CH376S chip itself; that requires surface mount soldering. 

If you've done soldering but are new to surface mounting chips: pin the chip into place by soldering two opposite corner pins, then with a hot iron and plenty of flux, run a bead of solder down each side, then wick up any excess.

### How to order for fabrication

Download [kicad/cocousbpak-fabrication.zip](kicad/cocousbpak-fabrication.zip), then upload it to one's PCB manufacturer when asked to provide Gerber files. Usually this is found under a 'Quote' option on the website. Search "pcb manufacturing" on any major search engine to get several manufacturers. NextPCB and PCBWay are two well-known ones.

Some may have ordered boards and have extra available. Reach out to don &#x40; dgb3.net to explore this.

Use the [Bill of Materials](kicad/cocousbpak.csv) to source the components from electronic supply houses such as Mouser, Jameco, or Digi-Key.

## Source and License

Design maintained at [https://github.com/barberd/cocousbpak](https://github.com/barberd/cocousbpak). [Kicad](https://www.kicad.org/) and [Freerouting](https://github.com/freerouting/freerouting/) were used to design the board.

The design is copyright 2025 by Don Barber. The design is open source, distributed via the GNU GPL version 3 license. Please see the COPYING file for details.

## Why

In 2022 I developed the [CoCo USB Serial Pak](https://github.com/barberd/cocousbserial), a cartridge that looks like an original Tandy Deluxe RS-232 pak to the CoCo but instead presents as a USB serial device to the PC its plugged into. That cartridge allows for very fast and easy communication between the CoCo and PC (think of it as a null modem cable), but does not provide any ability to host USB devices, to the confusion of many in the CoCo community.

After that project, I wanted to produce a cartridge that did provide that USB host ability for the CoCo. I also didn't want to just interface to another computer (such as a Raspberry Pi Pico or ESP) that does the real USB work; I wanted the CoCo to control the USB bus and communicate with devices directly, recognizing I'd have to write software drivers from scratch. This is the result.

## Using the Board

Please do not expect to use this board as a drop-in to replace the Color Computer's keyboard, joysticks, storage mediums, or other hardware and expect all the original software to work immediately and transparently.

This board provides a USB bus for the Color Computer. Use of that bus, and any devices found on that bus, requires software drivers. Patches for the original DECB routines for POLCAT (read a character from the keyboard) and JOYIN (sample the joysticks) to use USB keyboards and mice are available in the Software directory. These DECB patches work with most Basic programs and any machine language software coded to use DECB's built-in routines.

However, the majority of original applications and games are hard-coded to access the keyboard, joystick, and serial devices directly through the Peripheral Interface Adapter (PIA) chips of the CoCo and not through DECB's provided routines. If one desires, one can patch such software to instead leverage USB devices through this cartridge, but each software package is a custom effort. Routines to get started with this are available in the Software directory.

The exception for needing to patch software is when software is running under an operating system (where hardware is abstracted away through an API) and that operating system has USB drivers; the operating system's programming interface in combination with drivers takes care of the hardware interfacing. See the "NitrOS-9" section below.

To digress from this cartridge slightly: If one wants to use USB devices transparently with old software without patching, one will instead need a hardware solution that plugs directly into the original hardware's interface. This is normally done with a second computer (such as a Arduino, ESP, or Raspberry Pi Pico) doing the USB communication, converting the state information, and then presenting it via the legacy interface (such as plugging into the keyboard matrix ribbon connector plug or the joystick port). See [USB to CoCo Keyboard Adapter](https://thebreakkey.com/projects.html), [PiKey](http://www.pikey.tech/), and [CocoJoyStickAdapter](https://github.com/malfunct/CocoJoystickAdapter) for examples. The [CoCoSDC](http://cocosdc.blogspot.com/) uses a similar technique (though for an SD card interface, not a USB device). All these solutions are brilliant; they provide drop-in solutions requiring no change to legacy software.

The intent of this cartridge is different: unlike these hardware-interfacing solutions, this USB cartridge provides a full USB bus to the CoCo. This means by writing new drivers, one can use any USB device, including those not originally available on the CoCo. This could include scanners, printers, webcams, network adapters, bluetooth adapters, light guns, sound adapters, etc; almost any USB device can be made to work. This provides diverse flexibility for the CoCo developer to write new software leveraging new hardware; homebrew is still alive and well for the CoCo!

Two limitations to be aware: 

  * The CH376 chip does not support isochronous USB transfers (this means some of the streaming protocols often used on newer webcams won't work, though many webcams still support the interrupt and bulk transfer methods as well) 
  * The CoCo itself might be too slow for some USB devices - its bus speed can only transfer data so quickly.

### Configuration

Set the 7 dip switches (SW1) for the desired base IO address. These correspond to address lines A1 through A7. The default is $FF86. See [here](https://www.cocopedia.com/wiki/index.php/External_Hardware_IO_Address_Map) for a list of known IO hardware addresses. If you happen to have a conflict, choose a different IO address, set SW1 accordingly, and reassemble binaries to match.

The orientation of SW1 matters; it should be that the 'circuit is closed' or 'on' direction points to the 0 side. This is because the 74LS682 chip works by defaulting to '1' unless pulled to ground to make it a '0'. This simplifies and lowers the cost of the address selection circuit but also results in the non-intuitive use of 'on' to get a '0' instead of a '1'.

As such, make sure to orient the switch to match 'on' to face the same direction as the '0' label. If one gets the switch upside down, not a big deal, the switch will just operate in reverse from the silkscreened labels.

### Overview

The cartridge uses two ports, the base address and the base address+1. The two addresses used correspond to different registers on the CH376 and board. For example, if given base address of $FF86:

  * $FF86 Data Register (Read and Write)
  * $FF87 Command Register (Write) and Flag Register (Read)

Read the [CH376 Datasheet 1](datasheets/CH376DS1.PDF?raw=true) and [CH376 Datasheet 2](datasheets/CH376DS2.PDF?raw=true) for how to use the chip.

The CH376 chip has built-in commands to access USB mass storage devices both at the block level and file level. The CH376 also allows for control transfers (used to configure devices), interrupt transfers (used for keyboards, mice, game controllers, etc), and bulk storage transfers (used for mass storage such as pendrives). It does not support isochronous transfers (used for streaming data), so it will not work on modern webcams.

Note that the CH376 can only connect directly to a single USB device. That device can be a USB Hub, but this means driver software must configure the hub to allow communication to other devices through it. Such a driver can be found in the Software directory.

### Interrupts

The CH376 will issue an interrupt after completing several commands. This will assert the CART\* line on the cartridge. When PIA1 is configured to do so, a fast interrupt request is sent to the CPU when CART\* goes low.

To configure the PIA1 to do this, enable fast interrupts and configure it to trigger on a falling edge, like so:

                LDA     $FF23           ;load PIA1 Side B Control Register
                ORA     #$01            ;set bit 0 to 1 to enable cart firq
                ANDA    #$FD            ;set bit 1 to 0 to trigger on falling-edge
                STA     $FF23           ;store in PIA1 Side B Control Register

Hook in the interrupt handler at $010F (and/or $FEF4 on the CoCo3) as one would any other FIRQ handler. Make sure the interrupt handler performs a GET_STATUS to clear the interrupt.

If one's software doesn't handle interrupts, one will need to use software polling of the flag register instead.

See the routines in the Software directory for examples.

### ROM Socket

A socket for a ROM is also included in the board design. This is entirely optional, as many users today will instead choose to use another boot device, such as disk, SDC, or DriveWire. However, one can add a ROM loaded with a bootloader, perhaps to boot NitrOS-9 directly off a USB drive (see the "NitrOS-9" section below) or patching BASIC for USB drives and keyboards.

The ROM socket is wired for a 27128 EPROM. As the 27128 provides 16k of storage, the board presents this as 2 different banks of 8k each, selected via jumper J1. As such, when writing the EPROM, combine the images with each image aligned to an 8k boundary (eg, at 0 and 8k). If using a ROM IC other than the 27128, one will need to adjust the board design to match the chosen IC. Some other ROM ICs are pin-compatible and will work fine with the existing design.

Note Jumper J2, labeled "Autostart ROM". Color Computer cartridges signal the onboard ROM that they are present and should be started on boot by shorting the Q signal to the CART\* signal; see [Tandy Cartridge Schematic](https://colorcomputerarchive.com/repo/Documents/Manuals/Hardware/Color%20Computer%20ROM%20Cartridge%20Schematic%20%28Tandy%29.pdf). This works great for cartridges that don't make use of interrupts and makes it easy for end users to play cartridge games, but this technique conflicts for cartridges which need to send interrupts. This is evident in cartridges like the [Tandy Deluxe RS-232 pak](https://colorcomputerarchive.com/repo/Documents/Manuals/Hardware/Deluxe%20RS-232%20Program%20Pak%20Schematic%20%28Tandy%29.pdf), wherein users had to execute the ROM manually by running "exec &hc000" (Color Computer 1 or 2) or "exec &he010" (Color Computer 3) to start their software.

This cartridge attempts to remove this manual step from the user while still allowing the CH376 to send interrupts through the use of a flipflop and some additional logic: if the J2 jumper is present, then on startup, the Q signal is sent to CART\*, like any autostart cartridge. However, once the ROM is accessed, this signal is gated off, so normal interrupts from the CH376 can be sent. This works perfectly if booting from the cartridge and allows for both autostart and use of the CH376 interrupts.

However, if one has the J2 jumper present but boots via another method (perhaps via a floppy drive or a CocoSDC in another multi-pak interface slot), then the ROM is never accessed and the Q signal will continue to be asserted to CART\*. This will prevent proper use of interrupts. If one is not using the autostart feature, remove the J2 jumper to prevent this contention.

If one never plans on using the ROM, one can eliminate components J1, J2, U7, U5, R4, and Q1.

## More Info

### NitrOS-9

[NitrOS-9](https://github.com/nitros9project/nitros9) is a community maintained open source operating system for the Color Computer based on Microware's OS9. Several modules have been written for NitrOS-9 providing a USB Manager and drivers for various USB devices (such as mass storage devices, keyboards, and mice). Additionally, a boot module is available that boots NitrOS-9 directly off a pen drive connected to this cartridge when written to a ROM in the ROM socket.

### Links

* [CH376 Datasheet 1](datasheets/CH376DS1.PDF?raw=true)
* [CH376 Datasheet 2](datasheets/CH376DS2.PDF?raw=true)
* [USB 2.0 Specification](https://www.usb.org/document-library/usb-20-specification) Useful for various control transfer protocols
* [USB HID Specification](https://www.usb.org/sites/default/files/documents/hid1_11.pdf) Useful Keyboard and Mouse boot protocols
* [USB Mass Storage Specification](https://www.usb.org/sites/default/files/Mass_Storage_Specification_Overview_v1.4_2-19-2010.pdf)
* [USB Mass Storage Bulk-Only Transport](https://www.usb.org/sites/default/files/usbmassbulk_10.pdf)
* [SCSI Command Reference (from Seagate)](https://www.seagate.com/files/staticfiles/support/docs/manual/Interface%20manuals/100293068j.pdf) USB Mass storage devices use encapsulated SCSI
* [Color Computer Technical Reference](https://colorcomputerarchive.com/repo/Documents/Manuals/Hardware/Color%20Computer%20Technical%20Reference%20Manual%20%28Tandy%29.pdf) See Page 18 of the manual/25 of the PDF for bus timings.

### Note on the CH375B versus CH376S

The CH375 is a very similar chip that provides the same USB functionality of the CH376. The CH376 also costs slightly more and its additional features (interfacing with an SD card controller) are not used on this cartridge. Therefore, one might be tempted to use a CH375 instead. 

In the author's experience, the CH375 has an undocumented bug that drops a byte seemingly at random, leading to corrupted data and trouble keeping software and hardware in sync. This behavior was consistent across four chips the author tested, and seems consistent with anecdotal user experiences found online around other solutions using the CH375, such as ISA cards designed for early PCs.

If one does choose to use the CH375B instead, either add a bodge wire or modify the design before manufacturing to connect pin 23 to ground, and use $2B (WR_USB_DATA7) for the write command instead of $2C (WR_HOST_DATA).

## Errata

None at this time.

