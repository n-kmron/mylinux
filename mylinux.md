# Congratulation !

You're now able to start your own Linux kernel !

Let's start...

We won't use any packet manager from now, we'll downloads sources into `/usr/src`

First before, we need to mount our partition

* `mount /dev/sda1 /efi`
* `mount /dev/sda2 /boot`

1) Download sources (on /dev/sda4)

* `glibc` let's use version 2.39 (the same as the gentoo VM (see `ldd --version`) 
```
wget http://ftp.gnu.org/gnu/libc/glibc-2.39.tar.gz
```

* `kernel` in version 6.6.15
```
wget https://mirrors.edge.kernel.org/pub/linux/kernel/v6.x/linux-6.6.15.tar.gz
```

* `busybox` in version 1.34.0
```
wget https://busybox.net/downloads/busybox-1.34.0.tar.bz2
```

2) Create our FHS

* `cd /mnt` && mkdir mylinux`
* `mkfs.ext4 /dev/sda5` to make sda5 as ext4 fs
* `mount /dev/sda5 /mnt/mylinux`

* `mkdir -p /mnt/mylinux/{bin,boot,dev,etc,home,lib64,mnt,proc,root,sbin,sys,tmp,usr,var}`
* `mkdir -p /mnt/mylinux/usr/{bin,include,lib64,local,sbin,src,share}`
* `mkdir -p /mnt/mylinux/usr/localcd make`
* `mkdir -p /mnt/mylinux/usr/share/man`
* `mkdir -p /mnt/mylinux/usr/share/man/{man1,man2,man3,man4,man5,man6,man7,man8}`
* `mkdir -p /mnt/mylinux/var/{lock,log,run,spool}`

* `ln -s /mnt/mylinux/usr/share/man man` create a software link between

3) Glibc

* `tar -xvf glibc-2.39.tar.gz` extract sources
* `CFLAGS=”-O2 -U_FORTIFY_SOURCE -march=x86-64 -pipe” ../glibc-2.39/configure --enable-addons --prefix=/usr --with-headers=/usr/include` to generate the makefile in build folder

Now let's compile the sources 
* `date` to have the starting time
* `make -j2; date` to compile and have the ending time

For me, it started at 15:39:22 and finished at 15:50:00 so a total time of 00:20:38

To finish the installation, we have to install our glibc into `/mnt/mylinux`

* `make install_root=/mnt/mylinux install


4) Busybox

* `tar -xvjf busybox-1.34.0.tar.bz2` to extract sources (xvjf because its .bz2)
* `cd busybox-1.34.0 && make menuconfig` to save the config (if the compilation fail, remove utilitaries which create conflicts
* `make -j 2` to compile sources
* `make CONFIG_PREFIX=/mnt/mylinux install`

5) Creation configuration's files into `/mnt/mylinux/etc`

* `cd /mnt/mylinux/etc`
* `nano passwd` needs to contain : 
```
root::0:0:root:/root:/bin/ash
```
* `nano fstab` needs to contain :
```
/proc /proc proc defaults
```
* `/mnt/mylinux/bin/busybox dumpkmap > mykeyboard.kmap` to save our keyboard disposition
* `nano /mnt/mylinux/etc/rc` to create a boot script with : 
```
#!/bin/ash
/bin/mount -av
/bin/hostname YOURNAME
/sbin/loadkmap < /etc/mykeyboard.kmap
```

* `nano inittab` needs to contain :
```
::sysinit:/etc/rc
tty1:3:respawn:/bin/ash -l
```

6) Kernel

* `tar -xvf linux-6.6.15.tar.gz && cd linux-6.6.15` to extract sources
* `make defconfig` to configurate with default values
* `make menuconfig` add same configuration as gentoo kernel (see instructions.md)
* `date` to have the starting time
* `make -j 2; date` to compile and have the ending time
* `make modules_install` to install modules related to our kernel

For me, it started 16:36:04 at and finished at 17:01:25, so a total time of 00:25:21


Now, we can copy this kernel into our boot partition and giving a name

* `cp /usr/src/linux-6.6.15/arch/x86/boot/bzImage /boot/kernel-linux-6.6.15-NOUPOUE`

We now need to update the grub (you'll maybe need to create /grub folder)

* `grub-mkconfig -o /boot/grub/grub.cfg`

And now, we have to change the `grub.cfg` file to allow the OS on /dev/sda5:

* `/kernel-6.6.15-NOUPOUE root=/dev/sda5 console=tty0 console=ttyS1 rootfstype=ext4 rw`

And we need to activate in the VM settings the tty1 console : when the VM is shutted down > VM > Settings > Serial port > use output file (put a path on your physical machine)

You can now reboot and you should have your new kernel to use !

If the config when you launch your new boot is not right : actually no hostname, wrong keyboard, it's probably due to your `etc/rc` script, verify rights and execute it !
