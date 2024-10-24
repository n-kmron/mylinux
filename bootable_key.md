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

## Copy the RAMdisk and kernel

First, download `vmlinuz-generic` and `initrd.img`. After, send them to the VM using scp. Thos files are a linux extended kernel.

* `scp initrd.img-6.2.0-32-generic root@<vm_ip>:/home`
* `scp vmlinuz-6.2.0-32-generic root@<vm_ip>:/home`

When it's done, move them into sdb1 (so /mnt/usb-boot)

* `mv /home/initrd.img-6.2.0-32-generic /mnt/usb-boot/EFI/BOOT/`
* `mv /home/vmlinuz-6.2.0-32-generic /mnt/usb-boot/EFI/BOOT/`

Now, we'll download sources from syslinux

* `cd /home`
* `wget http://www.kernel.org/pub/linux/utils/boot/syslinux/syslinux-6.03.tar.gz`
* `tar -xvzf syslinux-6.03.tar.gz && rm syslinux-6.03.tar.gz && cd syslinux-6.03`
* `find ./ -type f -regex '.*\(syslinux\.efi\|ldlinux\.\(sys\|e64\|c32\)\)` to find files we need

And copy the files just found into /EFI/BOOT

* `cp ./efi64/efi/syslinux.efi /mnt/usb-boot/EFI/BOOT/bootx64.efi`
* `cp ./efi64/com32/elflink/ldlinux/ldlinux.e64 /mnt/usb-boot/EFI/BOOT`
* `cp ./bios/com32/elflink/ldlinux/ldlinux.c32 /mnt/usb-boot/EFI/BOOT`
* `cp ./bios/core/ldlinux.sys /mnt/usb-boot/EFI/BOOT`
* `blkid`

We can create a `syslinux.cfg` file into /EFI/BOOT

* `nano /mnt/usb-boot/EFI/BOOT/syslinux.cfg`
> ### Use the UUID of sdb2 (mylinux on USB) obtained with `blkid`
> PROMPT 0

> TIMEOUT 10

> DEFAULT YOURNAME

> LABEL YOURNAME

>    LINUX vmlinuz-6.2.0-32-generic

>    APPEND rw root=UUID=...

>    INITRD initrd.img-6.2.0-32-generic

We are now ready to copy our kernel root on the second partition of our usb driver
* `cp -ra /mnt/mylinux/* /mnt/usb-mylinux`

Everything is good now, let's unmount everything.

WARNING : please do a snapshot before !

* `umount -l /mnt/mylinux`
* `umount -l /mnt/usb-mylinux`
* `umount -l /mnt/usb-boot`

And now, let's try our key! 

* `poweroff`
