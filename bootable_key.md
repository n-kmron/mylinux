# Bootable key !

In this file, we'll see how to boot our kernel on a externel device (usb key)

First, we need to find which block device corresponds to the USB drive
* `lsblk` -> in my case, the 32GB USB drive appeared as /dev/sdb

## Partition the USB drive (GPT partition table)
* `su root`
* `fdisk /dev/sdb`
> Pass in GPT mode : g
> Create partitions and change type : n, t (type EFI:1, type Linux root x86-64:23, type Microsoft data:11)
> Save and exit : w

## Filesystem FAT32 on /dev/sb1

Download libraries
* `emerge --ask dosfstools`
* `emerge --ask ntfs3g`

Format the USB drive partitions
* `mkfs.fat -F 32 /dev/sdb1`
* `mkfs.ext4 /dev/sdb2`
* `mkfs.ntfs -f /dev/sdb3`

Mount the USB drive 
* `mkdir /mnt/usb-boot && mount /dev/sdb1 /mnt/usb-boot`
* `mkdir /mnt/usb-mylinux && mount /dev/sdb2 /mnt/usb-mylinux`
