BEFORE DOING THIS, BE SURE TO DO NOT FORGET PREREQUISITES (README.md)

BE SURE TO BE CONNECTED TO THE INTERNET

Once the VM is booted, you can start enter commands to build your system

## COMMANDS 

* `loadkeys be-latin1` changes the keyboard to match with yours
* `passwd` to set a password
* `useradd -m -G users,wheel user1` creates the first user and put him in 'users' and 'wheel' groups ('wheel' is to allow the ssh protocol for this user)
* `passwd user1` creates a password for our new user

Will launch a SSH session to have a better environment to start (to have the copy-paste etc)
* `/etc/init.d/sshd start` launch a SSH session
* `ip addr` get the VM's IP address

If you have a mistake at this step, verify that your computer is connected to internet.
If still, you should try this : `ip link ens33` then `ip link`

When you get the IP, you can, from another terminal on your physical machine, connect with SSH to the VM

* `ping <VM's IP>` to verify everything is OK
* `ssh user1@<VM's IP>` etablish connection

Now, we'll install the Gentoo library on our Kernel

Get your ISO prerequisites and verify their sha256sum authenticity

After that, check your disk with `lsblk` (should be `/dev/sda/`)

* `su -` to pass in admin mode
* `fdisk /dev/sda` to make our partitions

Now, in the fdisk console :
* `g` to create a GPT partition
* `n` to create our first part (100MB for UEFI) +100M
* `p` to see what it looks like -> we see that the type is 'Linux filesystem' but we want UEFI so let's change it
* `t` to change the type (L to see the type's list) to the number 1
* `p` we can see now the type is UEFI/EFI
* `n` create our second part (200MB for the boot) +200M
* `n` create our third part (2GB for the swap) +2G
* `t` change the type of the third part to Linux swap (number 19)
* `n` create our fourth part (20GB fir gentoo) +20G
* `t` change the type to linux root (number 23 - linux root (x86_64 because we're in a 64bit environment)
* `n` create our last part (remaining size for our linux)
* `w` to save everything 

Our partitions are created

We can check it by doing `fdisk -l` and `lsblk`

Let's create the Filesystem for our OS
We'll doing a FAT32

* `mkfs.vfat -F32 /dev/sda1`
* `mkfs.vfat -F32 /dev/sda2`
* `mkswap /dev/sda3`
* `mkfs.ext4 -T small /dev/sda4` the partition to compile our kernels (the T small option announce that we have a lot of small files so allows a bigger number of files because we cannot have more files that the number defined at the creation.

Mount the partition

* `mount /dev/sda4 /mnt/gentoo` to mount the 4th part of sda to a folder in the system. Without mounting folders on the hard disk, the data we provide is temp (in RAM). So, if we reboot, we lose our data. Now, our data on /mnt/gentoo is permanent.
* `cd /mnt/gentoo`
* `mkdir boot` create a folder to mount the boot (create the entry to the second partition)
* `mount /dev/sda2 /mnt/gentoo/boot` 
* `mount` recap of our mount's operations
* `mkdir /mnt/gentoo/efi` to create a folder for the UEFI
* `mount /dev/sda1 /mnt/gentoo/efi`

* `swapon /dev/sda3` to activate the swap

Now, we'll install the stage3 components which install /dev, /etc, ... dans /mnt/gentoo to have a complete system ON THE HARD DISK (and not in the RAM because now we have mounted partitions).

rem: in a shell, we can use `scp <filename> <user>` to copy a file to another user on the computer

* `scp <stage3filename.tar.xz> user1@<VM's IP>:/home/user1` do it from another terminal to copy the file from our computer to the /home/user1 repository on our VM

Go back on the VM, the file is temp. because /home is not mounted

* `mv /home/user1/<stage3filename> /mnt/gentoo`

The following command serve to extract the archive. Command found on the [gentoo handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Stage#Installing_a_stage_file)

* `tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner` 
* `rm <stage3filename>` delete the archive

* `cp --dereference /etc/resolve.conf /mnt/gentoo/etc`

We have to mount the stage3 on our system by execute the commands provided by the [gentoo handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base#Copy_DNS_info) 

* `mount --types proc /proc /mnt/gentoo/proc`
* `mount --rbind /sys /mnt/gentoo/sys`
* `mount --make-rslave /mnt/gentoo/sys`
* `mount --rbind /dev /mnt/gentoo/dev`
* `mount --make-rslave /mnt/gentoo/dev`
* `mount --bind /run /mnt/gentoo/run`
* `mount --make-slave /mnt/gentoo/run`


We have now a complete system on our root : /mnt/gentoo
The packet gestionnary is ~imerge?, so we can install nano, vim etc with it

WARNING : le root / is in the RAM so we have to change it

The [gentoo handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base#Entering_the_new_environment) give the commands to do it

```
chroot /mnt/gentoo /bin/bash
```
```
source /etc/profile
```
```
export PS1="(chroot) ${PS1}"
```

To save our work, we have to create a snapshot ! `VM (toolbar) > snapshot > take snapshot`


We have to chose a profile (systemd)

* `emerge-webrsync`
* `eselect profile list` to have the list of available profiles (our is 22)
* `eselect profile set 22` set the systemd profile as default

Let's change the timezone
* `ls -l /usr/share/zoneinfo/Europe` see if our timezone is available
* `ln -sf ../usr/share/zoneinfo/Europe/Brussels /etc/localtime` create a software link between the timezone and /etc/localetime

We also can custom our keyboard. We did it before, but it was only in RAM, let's do it on the disk

* `nano /etc/locale.gen` we have to keep an UTF8 keyboard so let's uncomment one of them and add a line like 
```
fr_BE.UTF-8 UTF-8
```

* `locale-gen` to generate binary files for our keyboards
* `eselect locale list` to see which keyboards do we have
* `eselect locale set 5` to set the 5th keyboard as the main one

Reload the new environment 
* `env-update && source /etc/profile && export PS1="(chroot) ${PS1}"`

Configure the kernel (source : [gentoo handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Kernel))

* `emerge --ask sys-apps/pciutils`
* `emerge --ask sys-apps/usbutils`
* `lspci -k` to see the hardware - it will be useful to select which drivers we'll need
* `lsusb` to see if we have devices connected on USB ports
* `lsmod`to see which drivers we already have on the livecd

WARNING : at the moment, the running kernel is still on the livecd, so in RAM

Get the sources from the gentoo's kernel

* `emerge --ask sys-kernel/gentoo-sources` to install sources into /usr/src

In Linux, when you chose to install an app without a package manager, you'll have to download sources as a compressed file, extract them, edit the config, compile the app to get executable and after, install it.

To compile and install our kernel from sources

* `cd /usr/src/linux-6.6.47-gentoo`
* `make menuconfig` to edit our config according to our needs (following the [hadbook](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Kernel#Alternative:_Manual_configuration) + some of personal changes)

In the configuration panel,
> Gentoo Linux > Support > support for init > systemd

> Device drivers > Generic driver opt > Maintain a devtmpfs filesystem to mount at /dev

> skip NMVE (we don't have it)

> Enable block layer > partition type > advanced > EFI

> File system > DOS/FAT/MSDOS > check the 2 NTFS

> Device drivers > Firmware drivers > EFI > check the following...
    - EFI Runtime config interface table version 2
    - EFI bootloader control 
    - EFI capsule loader 
    - EFI Runtime service tests support


> Device drivers > Fusion MPT > check Fustion MPT SCI Host drivers for SPI

> Device drivers > Graphic support > Framebuffer devices > enter > EFI based

> Device drivers > Graphic support > Framebuffer devices > support for frame buffer device > EFI based frame buffer support (enable it)

> Device drivers > SCSI device support > SCSI low-level drivers > VMware PVSCSI driver support

> Device drivers > MISC devices > VMware MCI driver

>  Processor type and features > EFI runtime service support > EFI stub support > disable (space)

At the end of our configuration, it's important to verify if "FUSION MPT" is on

* `cat .config | grep FUSION` and check if we see 'CONFIG_FUSION=y'

Now, we're able to compile our kernel (the better is to write the time before and after to have the total time of the compilation)

This compilation will product object's files and create more than 10GB and then reduct to an executable of 10MB.

* `date` to have the date before
* `make -j 2 ; date` to compile with 2 cores and at the end, get the date

For me, complilation started at 15:54:40 and done at 16:23:35
For a total time of : 00:28:45


We now have the kernel exe in /usr/src/linux-6.6.47-gentoo/arch/x86/boot/bzImage

Let's configure the fstab to define partitions, disks, swap etc automaticly on the reboot ([handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/System#About_fstab)).

* `emerge sys-kernel/installkernel`
* `emerge dracut`

* `nano /usr/lib/kernel/install.conf`
    -> layout=grub
    -> initrd_generator=dracut
    -> don't touch the rest

* `nano /etc/fstab`
    -> /dev/sda4    /   ext4    defaults,noatime    0 1
    -> /dev/sda3    none    swap    sw  0 0


We can give a name to our kernel

* `echo NOUPOUE2024 > /etc/hostname`

We have to create a DHCP client

* `emerge dhcpcd`
* `passwd`

Let's configure the bootloader ([handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Bootloader#Default:_GRUB))

Compile GRUB

* `emerge sys-boot/grub`
* `echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf`

Install GRUB

* `grub-install --target=x86_64-efi --efi-directory=/efi --removable`
* `cd /efi/EFI/BOOT` see if there is a file
* `cd /boot/grub` see if there is some files


We can use an utilitary to make a config

* `cp /usr/src/linux-6.6.47-gentoo/arch/x86/boot/bzImage /boot/kernel-6.6.47-NOUPOUE`
* `uname -a`
* `grub-mkconfig -o /boot/grub/grub.cfg`

Do a snapshot before reboot !

Let's reboot the system ([handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Bootloader#Rebooting_the_system))

* `exit`
* `cd`
* `umount -l /mnt/gentoo/dev{/shm,/pts,}`
* `umount -lR /mnt/gentoo`
* `reboot`
