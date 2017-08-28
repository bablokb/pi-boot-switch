Simple Usage of pi-boot-switch
==============================

Prerequisites
-------------

This tutorial assumes that you installed your Raspbian system to a SD-card
and configured it to your taste. It also assumes that you have a partitioned
usb device (e.g. a HDD, SDD or an usb-stick) with three partitions:

    $ > fdisk -l /dev/sda

    Disk /dev/sda: 7,5 GiB, 8011120640 bytes, 15646720 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x2052474d
    
    Device     Boot   Start      End Sectors  Size Id Type
    /dev/sda1          2048   165887  163840   80M  c W95 FAT32 (LBA)
    /dev/sda2        165888  8554495 8388608    4G 83 Linux
    /dev/sda3       8554496 15646719 7092224  3,4G 83 Linux

The first (small) partition will be our future boot partition. The other
partitions will serve as system-partitions for two installations. They
should be at least 2-8GB large, depending on your needs. For Raspbian-Lite,
2-4GB will do fine, for a full desktop installation you should at least
reserve 8GB. Anyhow, space should be no problem on modern HDDs/SDDs.

Note that you don't need to format the partitions, `pi-boot-switch`
can do this for you.

If you also want to boot from your usb-device, you should also prepare
*usb boot* following [this guide](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bootmodes/msd.md
"USB boot").

Personally I don't use usb boot, because it slows down the boot-process and
it is not reliable enough (too many usb devices are not supported). I keep
the boot-partition on the SD-card and the root-partition on usb. 


Partition Information
---------------------

The first step is to query partitition information using the option `-I`:

    $ > sudo pi-boot-switch -I
    Partition       | Label            | Role
    ----------------|------------------|---------------------
    /dev/mmcblk0p1  | boot             | current boot
    /dev/mmcblk0p2  |                  | current root / next root
    /dev/mmcblk0p3  |                  | 
    /dev/sda1       |                  | 
    /dev/sda2       |                  | 
    /dev/sda3       |                  | 

Partitions on the SD-card are named like `/dev/mmcblk0p1`, partitions
on the usb-device have a `/dev/sda` prefix (if you use multiple
devices in parallel, you will also see `/dev/sdb` and so on).

Since we booted from the SD-card, the current boot-partition should be
`/dev/mmcblk0p1` and the current root-partition is `/dev/mmcblk0p2`.
This will also be the next root-partition after a reboot.


Copy the system-partition
-------------------------

To copy the root-partition, run

    $ > sudo pi-boot-switch -c -t /dev/sda2 -F
    info: formatting partition /dev/sda2 with ext4
    info: mounting /dev/sda2 on /tmp/pi-boot-switch.FrSzev
    info: copying / to /tmp/pi-boot-switch.FrSzev
    info: fixing /etc/fstab on /dev/sda2
    info: copying /boot/ to /tmp/pi-boot-switch.FrSzev/_boot
    info: fixing /tmp/pi-boot-switch.FrSzev/_boot/cmdline.txt
    info: copied system to /dev/sda2
    info: use pi-boot-switch with option '-s /dev/sda2' to boot from new partition
    info: setting label for /dev/sda2: raspbian 8

This will copy all files from the current root-partition to the new
root-partition (passed in with the target-option `-t /dev/sda2`). Since we
also used the option `-F`, the script will format `/dev/sda2` with
the ext4-filesystem. If you already formatted your partition, just
omit this option.

You can also pass in the `-v` (verbose) option if you prefer
to have more output.

After the command has finished, run the info command again

    $ > sudo pi-boot-switch -I
    Partition       | Label            | Role
    ----------------|------------------|---------------------
    /dev/mmcblk0p1  | boot             | current boot
    /dev/mmcblk0p2  |                  | current root / next root
    /dev/mmcblk0p3  |                  | 
    /dev/sda1       |                  | 
    /dev/sda2       | raspbian 8       | 
    /dev/sda3       |                  | 

As you can see, not much has changed. The partition `/dev/sda2` now
has a label, but otherwise everything is as before.

There are two things to note: using `-F` destroys all existing data on
the target-partition, so be careful to select the correct partition.
Secondly, also without `-F` a copy operation is destructive. The
current root-partion is replicated to the target-partition and this
will overwrite all files and in addition automatically delete all
files on the target-partition which are not on the source-partition.

There is one exception if you also use the option `-k`: in this case,
the `/home`-directory is not copied to the target-partition. So you
keep data in the home-directories, but you would still loose all
configuration files e.g. in`/etc`. So make sure to backup all files
needed before you replicate.


Labelling the system-partition
------------------------------

The copy command will automatically label the selected partition, using
information found in `/etc/os-release`. Since we want a more meaningful
label, we change it with

    $ > sudo pi-boot-switch -t sda2 -L "production"
    info: setting label for /dev/sda2: production

    $ > sudo pi-boot-switch -I
    Partition       | Label            | Role
    ----------------|------------------|---------------------
    /dev/mmcblk0p1  | boot             | current boot
    /dev/mmcblk0p2  |                  | current root / next root
    /dev/mmcblk0p3  |                  | 
    /dev/sda1       |                  | 
    /dev/sda2       | production       | 
    /dev/sda3       |                  | 

Two things to note: the label is limited to 15 characters. Secondly, you
do not need to pass the `/dev/` part for the partition name. This is
not limited to labelling, `sda2` will always be a synonym for `/dev/sda2`.

We also label the root-partition on the SD-card. Since this is our
current root, we don't have to pass the `-t xxx` option:

    $ > sudo pi-boot-switch -L "original"
    info: setting label for /dev/mmcblk0p2: original

Switching the system-partition
------------------------------

Until now, we only copied the system partition from the SD-card to the
usb-device. The next command actually performs the switch:

    $ > sudo pi-boot-switch -s -t sda2
    info: copying /boot/ to /_boot
    info: mounting /dev/sda2 on /tmp/pi-boot-switch.FOT7cc
    info: copying /tmp/pi-boot-switch.FOT7cc/_boot/ to /boot
    info: unmounting /dev/sda2
    info: reboot the system to start from /dev/sda2

    $ > sudo pi-boot-switch -I
    Partition       | Label            | Role
    ----------------|------------------|---------------------
    /dev/mmcblk0p1  | boot             | current boot
    /dev/mmcblk0p2  | original         | current root
    /dev/mmcblk0p3  |                  | 
    /dev/sda1       |                  | 
    /dev/sda2       | production       | next root
    /dev/sda3       |                  | 

If you reboot the system now, it will boot from SD-card but the root-system
will be mounted on the partition `/dev/sda2`.

Note that switching the root-partition does not replicate user data. So
don't be surprised that files you created (e.g. in /home/pi) are no longer
there. They still exist, but on the original root-partition.

To replicate user-data you have to add the option `-u`. This will copy all
files from the `/home`-directory of the source-partition to the
`/home`-directory of the target partition.


Booting from the usb-device
---------------------------

If we also want to boot from the usb-device, we also need to copy the
boot-partition from the SD-card to the usb-device:

    $ > sudo pi-boot-switch -B -t sda1 -F
    info: formatting partition /dev/sda1 with Fat32
    mkfs.fat 3.0.27 (2014-11-12)
    info: mounting /dev/sda1 on /tmp/pi-boot-switch.GZGPMR
    info: copying /boot/ to /tmp/pi-boot-switch.GZGPMR
    fatlabel: warning - lowercase labels might not work properly with DOS or Windows
    info: fixing /boot in /etc/fstab on current root-partition 
    info: testing /dev/mmcblk0p1 ...
    info: /dev/mmcblk0p1 is not a relevant partition
    info: testing /dev/sda2 ...
    info: found _boot, fixing /boot in /etc/fstab on partition /dev/sda2
    info: testing /dev/sda1 ...
    info: /dev/sda1 is not a relevant partition
    info: activate USB-boot to and remove SD-card to boot from /dev/sda1

The option `-F` will again take care of formatting `/dev/sda1`. **Note that
it is essential to first switch the system-partition and then copy
the boot-partition**. Otherwise when booting from the usb-device the system will
still try to mount the old root-partition from the (non-existent) SD-card.

The command also updates the `/etc/fstab` of all systems to mount the
new boot-partition.

At this point you can shutdown the system, disconnect power, remove the
SD-card and repower the system again. If you activated usb-boot as described
in the prerequisites and your system (and device) is supported,
then your system should boot normally. Running

    $ > sudo pi-boot-switch -I
    Partition       | Label            | Role
    ----------------|------------------|---------------------
    /dev/mmcblk0p1  | boot             | 
    /dev/mmcblk0p2  | original         | 
    /dev/mmcblk0p3  |                  | 
    /dev/sda1       | boot             | current boot
    /dev/sda2       | production       | current root / next root
    /dev/sda3       |                  | 

should confirm that you booted from `/dev/sda1` and your current root is
`/dev/sda2`.


Using a second system on the usb-device
---------------------------------------

At this point we have an up and running Raspbian system labelled "production".
One golden rule of IT is to never change a running system, at least not
without a good backup. For this reason we copy the system to a new
partition. This time we will use an all-in-one command to demonstrate that
we can do all the steps we did above with separate commands with a
single commandline:

    $ > sudo pi-boot-switch -c -t sda3 -s -L "test" -F
    info: formatting partition /dev/sda3 with ext4
    info: mounting /dev/sda3 on /tmp/pi-boot-switch.NPKDvD
    info: copying / to /tmp/pi-boot-switch.NPKDvD
    info: fixing /etc/fstab on /dev/sda3
    info: copying /boot/ to /tmp/pi-boot-switch.NPKDvD/_boot
    info: fixing /tmp/pi-boot-switch.NPKDvD/_boot/cmdline.txt
    info: copied system to /dev/sda3
    info: reboot the system to start from /dev/sda3
    info: copying /boot/ to /_boot
    info: mounting /dev/sda3 on /tmp/pi-boot-switch.xDgauR
    info: copying /tmp/pi-boot-switch.xDgauR/_boot/ to /boot
    info: unmounting /dev/sda3
    info: reboot the system to start from /dev/sda3
    info: setting label for /dev/sda3: test

    $ > sudo pi-boot-switch -I
    Partition       | Label            | Role
    ----------------|------------------|---------------------
    /dev/mmcblk0p1  | boot             | 
    /dev/mmcblk0p2  | original         | 
    /dev/mmcblk0p3  |                  | 
    /dev/sda1       | boot             | current boot
    /dev/sda2       | production       | current root
    /dev/sda3       | test             | next root

We use the option `-c` to initiate copying of the current root-partition
to the *target*-partition (option `-t sda3`). We also switch to `/dev/sda3`
using option `-s`. Finally, we label the target-partition using the
option `-L`. Since we also want to format the target-partition, we use
the option `-F`. Note that the order of the options is not relevant.

So now we have two working installations, one on `/dev/sda2` and one on
`/dev/sda3`. You can use `pi-boot-switch -s` to switch back and forth
between both versions. Don't forget to reboot in between, since
switching only changes the root-partition for the *next-boot*.


Updating a partition
--------------------

Let's assume you did some changes using the system on `/dev/sda3`
(labelled 'test'). Once you are satisfied you want to go productive. In this
case you can use the normal copy command (assuming you have booted the
system on sda3):

    $ > sudo pi-boot-switch -c -t /dev/sda2

Without the format option `-F` this will copy new or differing files.
**Note that this command will also delete all files on `/dev/sda2` which
are not on `/dev/sda3`. So be very careful that you don't loose any files
you have created e.g. in your home-directory or in `/etc`!!** If you add
the `-k`-option, user-data is not copied and the existing `/home`-directory
on the target-partition will not be touched.
