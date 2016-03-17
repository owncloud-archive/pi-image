# Ubuntu Core image creation tools for ownCloud

Contained herein are scripts used to create Ubuntu Core images for the ownCloud
Pi self-hosting device based on the Raspberry Pi 2. The images are based on the
stable release channel of Ubuntu Core 16.04. Note that while the channel is
named "stable," Ubuntu Core 16.04 is still very much in beta. These scripts are
very much in beta as well. Use them at your own risk! They could eat your system
if used incorrectly.

Note that these scripts have only been tested on Ubuntu Xenial (16.04).


## What do these do?

So here's the deal: the ownCloud Pi device consists of a Raspberry Pi 2 and an
external hard drive. An image is required for the rpi2's SD card as well as the
external hard drive. Three scripts are contained within this repository:

- `create-snappy-image`

    Create an initial, monolithic Ubuntu Core image, pre-loaded with the
    [ownCloud snap][1]. Typically this is then flashed directly to the SD card,
    but for this device we only want to use the SD card for booting, and have
    the write-heavy stuff on the external drive.

- `split-image`

    Take the monolithic Ubuntu Core image and split its partitions out into
    separate images-- a `bootable.img` for the SD card and a `writable.img` for
    the external hard drive.

- `write-and-expand`

    A helper script to write an image to a device and expand it to utilize all
    available space.


## How do I use them?

The first step is to generate the initial Ubuntu Core image. Note that it
requires sudo, and may prompt for your password:

    $ ./create-snappy-image

That will download all sorts of things that make up the image, and put it
together. After it's done you'll end up with `pi2.img`.

Next, we need to split that image into separate images, one for the SD card,
and one for the HD:

    $ ./split-image pi2.img

That will extract the partitions from the `pi2.img` and create two new images,
`bootable.img` and `writable.img`.

Finally, we write each image to its respective device. Note that each device
should first be unmounted from the system if necessary. Now, write
`bootable.img` to the SD card:

    $ sudo dd if=bootable.img of=/dev/mmcblk0 bs=32M
    $ sync

Now, write `writable.img` to the HD. Note, however, that the image is only a few
GBs. You could of course use `dd` again to place it on the disk, but that would
restrict your disk to the size of the image. In order to write the image _and_
expand to use all available space on the disk, use `write-and-expand` (note that
it also may prompt for your password):

    $ ./write-and-expand writable.img /dev/sdb
    $ sync

This step will take some time, especially if your disk is large. Be patient.


## Now what?

Now you insert the SD card into the rpi2, connect the external HD, connect it to
your router via an ethernet cable, and apply power! Once it comes up you should
be able to visit `owncloud.local` in your browser.

[1]: https://github.com/kyrofa/owncloud-snap
