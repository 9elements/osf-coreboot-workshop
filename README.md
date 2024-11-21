# OSF coreboot workshop

## Relevant web resources

- [ODROID H4 Wiki](https://wiki.odroid.com/odroid-h4/start)
- [coreboot documentation](https://doc.coreboot.org/)

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

Before starting the configuration, extract the relevant Flash regions using [ifdtool](https://doc.coreboot.org/util/ifdtool/binary_extraction.html).
The extracted soc information will be saved to your local working directory.

<details>
<summary>Intel FDT</summary>
The Intel Flash Descriptor (IFD) defines offsets and sizes of various regions of the flash.
It is used to define coreboots flashmap, which describes partitions on the flash chip.
For further details refer to the [official coreboot documentation](https://review.coreboot.org/plugins/gitiles/coreboot/+/0cd098e4e41d6bb3b27327d4a6526bd7004bfc77/Documentation/ifdtool/layout.md).
</details>

```
/path/to/ifdtool -x /path/to/ODROID-H4-PLUS-1.rom
```

The command above should dump the neccessary binaries into your current directory. The files will be called something like "flashregion_*_*.bin". Now we need to tell coreboot where the binaries are.

If you have built another mainboard before (e.g. QEMU), you should run `make distclean` before configuring the new target.  
For configuration start `make menuconfig`, select "Hardkernel" as the mainboard vendor and include the IFD (Flash descriptor) and ME (Management Engine) flash region via the config menu by their file path.  
IFD make menuconfig path: "Chipset/Add Intel descriptor.bin file".  
After enabling the option a new option should appear which needs a file path to the "flashregion\_0\_flashdescripttor.bin" file  
ME make menuconfig path: "Chipset/Add Intel ME/TXE firmware"  
After enabling the option a new option should appear which needs a file path to the "flashregion\_2\_intel\_me.bin" file  

### Select Payload (EDK2-Payload)

In the menuconfig choose: `Payload/Payload to add/edk2 Payload`  
In the menuconfig choose: `Payload/Tianocore's EDK 2 payload/edk2 Payload`  
In the menuconfig Insert `edk2-stable202408` into: `Payload/Payload to add/Insert a commits SHA-1 or a branch name`  

All other default configurations are sufficient to boot the board.

## Flash H4+ coreboot firmware

```
./flashprog -p serprog:dev=/dev/ttyACM2 -w PATH/TO/COREBOOT/build/coreboot.rom
```
Now disconnect the flash clip.

### Connect UART pins

The GPIO pin assignment of our target hardware can be found here:
[ODROID Wiki: H4 I/O Expansion GPIO MAP](https://wiki.odroid.com/odroid-h4/hardware/io_expansion_gpio)

Use a tree jumper cable to connect GND and RX to TX and vice versa.

## Boot coreboot

Depending on the configured log level, the coreboot console provides substitute information during
the boot process. Therefore, you should make sure that the UART is set up properly.
The [picoprog provides an additional interface to connect to the UART](https://github.com/9elements/picoprog?tab=readme-ov-file#uart-communication).
Use the software of your choice to connect to it. e.g:
```
<tio/picocom> -b 115200 /dev/serial/by-id/usb-9elements_Picoprog_OSFC2024-if02
```
or
```
cu -l /dev/serial/by-id/usb-9elements_Picoprog_OSFC2024-if02 -s 115200
```

### Verify UART Communication

We have live ubuntu Linux system with the serial console enabled by default on USB sticks. To test if the
UART communication is working, you should first boot the system and check. You should see coreboot logs as well as Linux ubuntu logs and a console to login into.

### Operating System

If you have the USB stick attached to the odroid, you should see a ubuntu live iso image booting up.
Congratulations you have successfully compiled, flashed and booted Firmware.

## Going Further

Now you can either go free style or try to solve some of the challenges that we have prepared. You could for example poke around with the code and the configuration options. Just be careful with the internal GPIO related configuration as a misconfiguration could potentially brick the hardware. If you run out of ideas, just ask the staff. They might know something you can try out.

### Challenges

TODO
