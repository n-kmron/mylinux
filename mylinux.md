# Congratulation !

You're now able to start your own Linux kernel !

Let's start...

We won't use any packet manager from now, we'll downloads sources into `/usr/src`

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

* `ln -s /mnt/mylinux/usr/share/man man` create a software link between ... and ...
