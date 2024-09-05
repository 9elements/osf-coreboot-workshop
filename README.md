# OSF coreboot workshop

## Relevant web resources

- [ODROID H4 Wiki](https://wiki.odroid.com/odroid-h4/start)
- [coreboot documentation](https://doc.coreboot.org/)

## Getting started with coreboot on QEMU

The coreboot documentation provides a tutorial on how to [start from scratch](https://doc.coreboot.org/tutorial/part1.html#tutorial-part-1-starting-from-scratch).
Follow the instructions there and try to boot QEMU with coreboot as firmware.

## Read H4+ vendor firmware

Before we can build a complete coreboot firmware image for the H4+, we need to read the manufacturer's firmware to extract the relevant flash regions.

### Connection between programmer and host firmware flash

The flash chip does not have a connector to access the flash chip. However, due to the SOI8 package
format, the pinout is easily accessible using the appropriate test clip. The only thing to note is
the polarity of the connection. Pin 1 (CS) is marked with a dot on the chip and a red wire on the
test clip, which must match to avoid bricking the board. The picture below shows the primary chip
we will use to flash coreboot.

![](/images/odroid-h4-ultra-flash-1.jpg)

### How to use the programmer

To use the picoprog to read the NOR flash containing the firmware, we need to use a programmer
software that supports the [Serprog Protocol](https://www.flashrom.org/supported_hw/supported_prog/serprog/overview.html).
For this we recommend the use of either [flashrom](https://www.flashrom.org/) or [flashprog](https://flashprog.org/wiki/Flashprog).
For detailed instructions on how to read the flash chip, please refer to the [picoprog README.md](https://github.com/9elements/picoprog?tab=readme-ov-file#usage).

## Build coreboot for ODROID H4

The patches to support the board are currently wip and must be obtained from coreboot gerrit:
```
git fetch https://review.coreboot.org/coreboot refs/changes/15/84215/1 && git checkout -b change-84215 FETCH_HEAD
```

Before starting the configuration, extract the relevant Flash regions using [ifdtool](https://doc.coreboot.org/util/ifdtool/binary_extraction.html).

```
/path/to/ifdtool -x /path/to/ODROID-H4-PLUS-1.rom
```

If you have built QEMU before, you should run `make distclean` before configuring the new target.
For configuration, except for selecting the motherboard and including the IFD and ME flashregion,
the default configuration is sufficient to boot the board.

## Flash H4+ coreboot firmware

You've already read the flash. I'm sure you know the drill. :)

## Boot coreboot

Depending on the configured log level, the coreboot console provides substitute information during
the boot process. Therefore, you should make sure that the UART is set up properly.
The [picoprog provides an additional interface to connect to the UART](https://github.com/9elements/picoprog?tab=readme-ov-file#uart-communication).
Use the software of your choice to connect to it. e.g:
```
<tio/picocom> -b 115200 /dev/serial/by-id/usb-9elements_Picoprog_OSFC2024-if02
```

### Connect UART pins

The GPIO pin assignment of our target hardware can be found here:
[ODROID Wiki: H4 I/O Expansion GPIO MAP](https://wiki.odroid.com/odroid-h4/hardware/io_expansion_gpio)

Use a tree jumper cable to connect GND and RX to TX and vice versa.

### Verify UART Communication

We have pre-installed a Linux system with the serial console enabled by default. To test if the
UART communication is working, you should first boot the system and check. But wait, we already
flashed the coreboot, right? This is where the secondary flash chip comes in handy. Use a jumper
to connect the pins to select it and boot the Linux system. The login credentials are `root:osf`.

## Going Further

Once you have successfully reached the coreboot payload stage, you are ready to begin your
adventures. Try out different payloads or play around with the configuration options. Of course, you
can also look at the code and poke around. Just be careful with the internal GPIO related
configuration as a misconfiguration could potentially brick the hardware. If you run out of ideas,
just ask the staff. They might know something you can try out.
