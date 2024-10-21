# Bootable key !

In this file, we'll see how to boot our kernel on a externel device (usb key)

First, we need to find which block device corresponds to the USB drive
* `lsblk` -> in my case, the 32GB USB drive appeared as /dev/sdb

## Partition the USB drive (GPT partition table)
* `su root`
* `fdisk /dev/sdb`
