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

