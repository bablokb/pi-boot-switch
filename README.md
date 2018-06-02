Pi USB Boot Switcher
====================

Introduction
------------

Installation of Raspbian (or other similar distributions) on a SD-card
is easy and straightforward. In contrast *running* the system directly
from the SD-card is not optimal, especially if you have a lot of
I/O-operations. A solution to the problem is to copy the installation
to an usb device, typically a HDD or SSD, but an usb-stick will also
boost performance (unless you happen to have a really slow one).

To run a system from an USB device, you have two options:

  - Keep the boot partition with firmware and kernel on the SD-card,
    but move the system partition (also called root partition) to the
    usb device. This solution works with all available models.
  - use *usb boot*. This is a new feature available with a Pi3. In this
    case the boot partition will also reside on the usb device and you
    don't need a SD-card at all. This only works with a Pi3 and even
    then not every usb device supports the operation.

The `pi-boot-switch`-script supports both options. The script allows you
to

  - copy your root-partition from the SD-card to SD-card.
  - copy your root-partition from the SD-card to the usb device.
  - copy your boot-partition from the SD-card to the usb device.
  - switch between multiple systems (a sort of bootmanager).
  - install new images directly to a target-partition.

For a quick walkthrough read the document
[doc/simple-usage.md](./doc/simple-usage.md "simple usage").


Installation
------------

To install the script, just run

    git checkout https://github.com/bablokb/pi-boot-switch.git
    sudo tools/install

This just installs the script in `files/usr/local/sbin/pi-boot-switch` to
`/usr/local/sbin/pi-boot-switch`.


Documentation
-------------

After installation, Raspbian usually expands it's root-filesystem and
claims all available space on the SD-card. A prerequisite for
`pi-boot-switch` is free space, so you have to shrink the filesystem
and partition to regain some of it. How to do that is covered in the
document
[doc/shrink-rootsystem.md](./doc/shrink-rootsystem.md "Shrinking the root-filesystem").

You can find a complete walkthrough of the most important options in the
tutorial [doc/simple-usage.md](./doc/simple-usage.md "simple usage").

Installing an additional image using the option `-i` is covered in
[doc/image-install.md](./doc/image-install.md "image installation").

Options not covered in these documents are either not yet implemented,
or for special use. Please check the code before using them.

For a short help just run

    $ > sudo pi-boot-switch -h

    pi-boot-switch: manage Raspberry Pi multiboot system
      
    usage: pi-boot-switch [options]
      Possible options:
    
        -I          show partition info
    
        -c          copy partition (select target with -t)
        -t dest     target partition (required for -i, -c or -s, e.g. '-t /dev/sdc')
        -C other    copy from another partition (select target with -t)
        -S          target partition is on a SD-card, not an usb-device
        -F          format target partition
        -k          keep existing /home on target during copy (don't use -F)

        -L label    set label of target partition (also available standalone)
    
        -B          copy /boot to partition part and set part as new /boot
    
        -s          switch to partition dest for next boot (requires reboot)
                    (select target with -t)
        -u          update /home on dest from current partition before switching
        -R          reboot immediately after switching
    
        -i image    install image to target partition
    
        -v          verbose operation
        -h          show this help


Armbian support
---------------

Pi-boot-switch now also supports Armbian. Since Armbian uses a single
partition for both the boot and root filesystem, you first have to
convert the image. You can download a suitable script from my
[armbian-convert project](https://github.com/bablokb/armbian-convert "armbian-convert-project"). This project contains a single file `armbian_convert.sh`,
which will convert a typical armbian 7z-image with a single partition
to an image with two partitions (one for boot and one for root).

Once converted, you can install the image like any other image with
e.g. `dd` to your sd-card and use it with `pi-boot-switch` to switch
between partitions.
