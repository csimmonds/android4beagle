# Android4Beagle

Android4Beagle aims to provide a vanilla AOSP-based Android for
Beagleboard.org boards, starting with the BeagleBone Black. Hopefully
I will get round to adding the other colours of BeagelBone later,
and maybe even the BeagleBoard-xM and BeagleBoard-X15. All of these
make great platforms for **embedded Android**, by which I mean Android being
used to control ... I don't know ... a printer, a robot, maybe some
kind of test or medical equipment.

If you would like to know how it is done, and how you can apply this
knowledge to your own products, then you might be interested in
what I do for my day-time job
[http://www.2net.co.uk/training.html](www.2net.co.uk/training.html)


## Board support matrix

|   Board        | AOSP  | Kernel | Cape manager | SGX | Tag              |
| -------------- | ----- | ------ | ------------ | --- |----------------- |
|BeagleBone Black| 4.4.4 | 3.2    | N            | Y   | android-4.4.4_r2 |
|                | 6.0.1 | 3.8    | Y            | N   | android-6.0      |

## Preparation

Make sure that you have a system capable of building AOSP in a reasonable
amount of time, as described here:
[http://source.android.com/source/building.html](http://source.android.com/source/building.html).
Then follow these steps to set it up given here:
[http://source.android.com/source/initializing.html](http://source.android.com/source/initializing.html)

You will need in addition the U-Boot **mkimage tool**. On Ubuntu run
`sudo apt-get install u-boot-tools`

## Getting Android4Beagle

The majority of the code comes directly from android.googlesource.com.
The only board-specific components are U-Boot, the Linux kernel, the PowerVR
SGX GPU driver and the scripts needed to put it all together.

Begin by getting the Android4Beagle repo, using the appropriate tag from the
product matrix:

```
$ mkdir ~/a4b
$ cd ~/a4b
$ repo init -u https://github.com/csimmonds/android4beagle -b android-6.0
$ repo sync -c
```
The googlesource repositories are always growing. The last time I checked,
the total download was 55 GiB, so please
make sure that you have enough disk space, bearing in mind that the
build will generate another 25 GiB of intermediate object files
and the final images.

# Building for BeagleBone Black
There are two product variants:

* beagleboneblack_sd-eng: if you want to run from a micro SD card
* beagleboneblack-eng: if you want to install Android in internal flash

## Build
```
$ source build/envsetup.sh
$ lunch
```
Select either **beagleboneblack-eng** or **beagleboneblack_sd-eng**. Then
```
$ ./build-beagleboneblack.sh
```
The build will take at least one hour, possibly much longer if
your build machine is a little below spec.

## Running Android from the micro SD card

You will need a micro SD card of at least 4 GB. Ideally it should
be class 10 or better.

Plug the card into your card reader. Run command `lsblk` to find which
device it is. Then run the script below, giving the device name as the
parameter. For example, if the card reader is `/dev/mmcblk0`, the
command would be:
```
$ scripts/write-sdcard-beagleboneblack.sh mmcblk0
```
When done, plug the card into your BeagelBone Black, press the
"boot" button and power up.

## Running Android from internal eMMC flash memory

In this case, you begin by making a bootable SD card which contains
only U-Boot. You boot from it, and then use fastboot to format
the eMMC chip and then copy the Android images to it.

Plug a micro SD card into your card reader. Run command `lsblk` to find which
device it is. Then run the script below, giving the device name as the
parameter. For example, if the card reader is /dev/mmcblk0, the
command would be
```
$ croot
$ scripts/write-fastboot-sdcard-beagleboneblack.sh mmcblk0
```
When done, plug the card into your BeagelBone Black, press the
"boot" button and power up.

The "fastboot LED" (user LED0) should come on, indicating that it
is in fastboot mode. Now use fastboot to program the internal flash:
```
$ croot
$ cd u-boot
$ fastboot oem format
$ fastboot reboot
```
This will force U-Boot to read the new partition table. When the
fastboot LED comes on again, continue:
```
$ fastboot flash spl MLO
$ fastboot flash bootloader u-boot.img
$ fastboot flash userdata
$ fastboot flash cache
$ fastboot flash system
$ fastboot flash boot
$ fastboot flash recovery
```
Power off, **remove the SD card** and power on. It should boot Android...

If something goes wrong which flashing the images, the best diagnostic
tool is to plug a serial to USB connector into the Beagle and see what
U-Boot is doing. You can force it into fastboot mode by typing the
command `fastboot`
