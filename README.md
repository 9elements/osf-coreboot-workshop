# osfc-odroid-workshop
OSFC ODROID H4 coreboot workshop

# How to
## Dumping firmware regions
```console
cd coreboot/util/ifdtool
make
./ifdtool -x stock-fw/ODROID-H4-PLUS-1.rom
```

## Building coreboot

### Retrieving our coreboot port for the ODROID H4
```console
cd coreboot/
git fetch https://review.coreboot.org/coreboot refs/changes/79/83979/3 && git checkout -b change-83979 FETCH_HEAD
```

### Configuring coreboot
```console
cp configs/odroid-h4.defconfig coreboot/
cd coreboot/
make defconfig KBUILD_DEFCONFIG=odroid-h4.defconfig DOTCONFIG=.config
```

### Compiling coreboot
```console
make -j$(nproc)
```

## Flashing the firmware
Once you've properly connected a flasher (dediprog, ...) to your ODROID, use tools like flashrom or dpcmd
to program the flash memory with the coreboot image

```console
./dpcmd --device 1 -u <path/to/coreboot.rom>
```

## Reading the serial
Connect your PC to the ODROID using an USB-to-serial adapter and use minicom (or alternatives) to read
the serial

```console
sudo minicom -b 115200 -D /dev/ttyUSB0 -C log.txt
```

## Booting the ODROID
Plug in the power supply and press, if necessary, the power button. The board should blink up red ***and*** blue
and serial logs should appear on your minicom connection.
