Raspberry Pi Autoplay
=====================

What is it?
-----------
This is a [Rodent Linux](http://rodentlinux.org) package with all the
configuration files needed to create a root filesystem for the Raspberry
Pi (1 or Zero) that simply boots up and begins playing videos from the
SD card.
It does this without ever writing to the SD card, so you can safely cut
the power at any time.

Furthermore the root filesystem itself is simply a file on the SD card,
so the card can be partitioned with just a single FAT32 filesystem, and
the root can be updated by writing a new file to the card.

Creating a root filesystem
--------------------------
You'll need a clone of this repository and a clone of the rodent repo.
```sh
git clone https://github.com/rodentlinux/rpi-autoplay.git
git clone https://github.com/rodentlinux/rodent.git
```

Set up your copy of rodent by creating a settings file
```sh
cd rodent
cp settings.proto settings
```

Since this only works a Raspberry Pi 1 or Zero we're only interested
in the 'rpi' port of Rodent. Just edit the settings file so the first
line only reads
```sh
ARCH=rpi
```

Now we can update the package repositories.
```sh
./ro update
```

To build the rpi-autoplay package just type
```sh
./ro make ../rpi-autoplay
```
This builds the package and updates your local repository of home built
packages in the `cache/local` subdirectory. The package just contains
plain text configuration files and symbolic links, so no (cross compiling)
toolchain is needed to build it.

We need to install the rpi-autoplay package package we just build and
all the dependencies to temporary directory.
```sh
./ro install rpi-autoplay
```
This will download and install a bunch of other Rodent packages which are
needed by our rpi-autoplay package. The downloaded packages will be stored
in `cache/rodent`. By default the packages will install to a directory
called `sysroot.rpi`.
You can inspect the installed files by browsing this directory.

Lastly we need to 'squash' all these files into something that can be used
as a root filesystem by the Raspberry Pi. We use squashfs for this. You can
think of it as a zip-file (or tarball) that can be mounted as a filesystem.
```sh
./ro squash
```
Congratulations! You've just created your first Rodent Linux root filesystem
in `image.sqfs`.


Preparing an SD card
-----------------------

Unfortunately the Raspberry Pi is a bit picky about how the SD is
formatted. If it is not done right it just won't boot. If in doubt you
can always use a working SD card, just copy the files to that and skip
this chapter.

The following guide works on Linux. If you know a way to do the same on
MacOS or Windows, please submit a pull request.

First you need to identify the block device of your SD card. This will
usually be something like `/dev/mmcblk0` or `/dev/sdb`. In the following
I'll call it `/dev/<sdcard>`, so just change that to your real block device.

WARNING: Be very careful about picking the right block device. If you run these
commands on the wrong block device you'll seriously mess up what is stored
there and probably loose all your data.

First run fdisk to create a new partition table readable by the Raspberry Pi
```sh
fdisk -H 255 -S 63 /dev/<sdcard>
```

Create a new empty DOS partition table by typing `o` (followed by `<enter>`).
Create a partition spanning the whole disk with
```
n, p, 1, , ,
```
where the commas are `<enter>`.
Mark the partition as a FAT32 filesystem with
```
t, c,
```
Lastly write and exit fdisk by pressing `w`.

Now the SD card is partitioned, but we still need to create the actual FAT32
filesystem.
```sh
mkfs.vfat /dev/<sdcard>1
```
Notice there is an extra `1` at the end of the block device. If your
SD card is `/dev/mmcblk0` the first partition is actually `/dev/mmcblk0p1`,
but if your SD card is `/dev/sdb` the first partition is just `/dev/sdb1`.
If you don't have that command you'll need to install the `dosfstools`
package.

Now you should be able to mount the FAT32 filesystem on your SD card
and copy files to it.


Copy everything to the SD card
------------------------------

In your copy of the `rpi-autoplay` repository is a directory called 'sdcard'.
Just copy everything in that folder to your card. This is the bootloader,
kernel and other configuration needed to boot the Raspberry Pi.

Also copy the `image.sqfs` root filesystem you created earlier to the card,
but call it `root-01.sqfs`. The kernel will look for files on
the SD card called `root-<something>.sqfs` and use whichever comes
last when sorted. This way you can try out another root filesystem without
overwriting the old one by just naming it eg. `root-02.sqfs`.

Lastly copy some h263 encoded videos to the `videos` folder, eject your
SD card, stick it in your Raspberry Pi and enjoy :)
