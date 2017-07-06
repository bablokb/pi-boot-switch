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

  - copy your root-partition from the SD-card to the usb device.
  - copy your boot-partition from the SD-card to the usb device.
  - switch between multiple systems (a sort of bootmanager).

For a quick walkthrough read the document
[doc/simple-usage.md] (./doc/simple-usage.md "simple usage").


Installation
------------

To install the script, just run

    git checkout https://github.com/bablokb/pi-boot-switch.git
    sudo tools/install

This just installs the script in `files/usr/local/sbin/pi-boot-switch` to
`/usr/local/sbin/pi-boot-switch`.


Documentation
-------------

Besides the tutorial in
[doc/simple-usage.md] (./doc/simple-usage.md "simple usage") there is no
extensive documentation, as all the options should be self-explantory.

For a short help just run

    $ > sudo pi-boot-switch -h

    pi-boot-switch: manage Raspberry Pi multiboot system
      
    usage: pi-boot-switch [options]
      Possible options:
    
        -I          show partition info
    
        -c          copy partition (select target with -t)
        -f source   source partition (default: current root-partition)
                    use with -c, e.g. '-f /dev/mmcblk0p2'
        -t dest     target partition (required for -i, -c or -s, e.g. '-t /dev/sdc')
        -F          format target partition
        -L label    set label of target partition (also available standalone)
    
        -B part     copy /boot to partition part and set part as new /boot
    
        -s          switch to partition dest for next boot (requires reboot)
                    (select target with -t)
        -R          reboot immediately after switching
    
        -i image    install image to target partition
    
        -v          verbose operation
        -h          show this help
