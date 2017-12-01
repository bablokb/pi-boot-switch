Install a Raspbian-Image to a target partition
==============================================

Pi-boot-switch also supports the direct installation of a Raspbian-image
to a given target-partition. To use this feature, you need at least a free
system-partition (used as installation target). The best procedure is
to download the image from raspberrypi.org and put it on an USB-stick.

The script accepts images compressed with zip (download-format). You do
not have to unzip it, but there must be enough free space on the
partition where the image file resides to hold the unzipped image.

Installation is straight forward, use a command like:

    sudo pi-boot-switch -i "path-to-image.zip" -t /dev/mmcblk0p3 -F

This command only installs the image, to switch to the newly installed
system, use the `-s` option. You can also use the `-L` option to
set the label for the new system-partition.

Note that this kind of installation of Raspbian is not supported by
the foundation, so the system is in some small details different from
a standard installation using dd or etcher.
