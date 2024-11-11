# Docker installation

In this file, we'll see how to install docker on our gentoo distribution to use it on our personal kernel after !

First, we have to configure our kernel with namespaces, cgroups, etc

WARNING: we now working on our Gentoo kernel to edit our personal one !

## Let's change the kernel configuration

* `ln -s /usr/src/linux-6.6.15 /usr/src/linux` to create a symbolic link and recompile our kernel

* `cd /usr/src/linux`

* `make menuconfig` and add these options (no modules (M), only \*) 

> General setup > BRF subsystem > Enable bpf() system call

> General setup > Control Group support > Memory controller + Support for eBPF programs attached to cgroups

> General setup > Control Group support > CPU controller > CPU bandwidth provisionning for FAIR_GROUP_SCHED + Group scheduling for SCHED_RR/FIFO

> General setup > Namespaces support > User namespace

> Enable the block layer > Block layer bio throttling support

> Networking support > Networking options > Networking packet filtering framework (Netfilter) > Advances netfilter configuration > Core Netfilter Configuration > TFTP protocol support

> Networking support > Networking options > Networking packet filtering framework (Netfilter) > Advances netfilter configuration > Core Netfilter Configuration > ***Xtables matches *** > "addrtype" address type match support

> Networking support > Networking options > Networking packet filtering framework (Netfilter) > Advances netfilter configuration > IP virtual server support > *** IPVS transport protocol load balancing support *** > TCP load balancing support + UDP load balancing support

> Networking support > Networking options > Networking packet filtering framework (Netfilter) > Advances netfilter configuration > Core Netfilter Configuration > ***Xtables matches *** > "ipvs" match support

> Networking support > Networking options > Networking packet filtering framework (Netfilter) > Advances netfilter configuration > IP virtual server support > *** IPVS scheduler *** > round-robin scheduling

 > Networking support > Networking options > Networking packet filtering framework (Netfilter) > Advances netfilter configuration > IP virtual server support > *** IPVS MH scheduler *** > Netfilter connection tracking

> Networking support > Networking options > Networking packet filtering framework (Netfilter) > Advances netfilter configuration > IP: Netfilter Configuration > iptables NAT support + MASQUERADE target support + NETMAP target support + REDIRECT target support

> Networking support > Networking options > IP: ESP transformation + 802.1Q/802.1ad VLAN support + 802.1d Ethernet Bridging > VLAN filtering

> Networking support > Networking options > L3 Master device support

> Device Drivers > Multiple devices driver support (RAID and LVM) > Device mapper support > Thin provisioning target

> Device Drivers > Network device support > Dummy net driver support + MAC-VLAN support + Virtual eXtensible Local Area Network (VXLAN) + Virtual ethernet pair device

> File systems > Overlay filesystem support

> Security options > Enable access key retention support > Enable register of persistent per-UID keyrings

> Security options > Enable access key retention support > ENCRYPTED KEYS

> Security options > Enable access key retention support > Diffie-Hellman operations on retained keys

## Compiling and setup the new version for our kernel 

* `date` to have the starting date

* `make -j 2 && make modules_install; date` to compile and have the ending date

For mine, the compilation started at 17:34:56 and ends at 18:14:08, so a total time of 00:39:12. 
* `lsblk` to remount our missing partitions

* `mount /dev/sda2 /boot`

* `cp /usr/src/linux/arch/x86/boot/bzImage /boot/kernel-6.6.15-NOUPOUE-DOCKER` to place a new kernel on our boot

* `grub-mkconfig -o /boot/grub/grub.cfg` to edit our grub.cfg

* `nano /boot/grub/grub.cfg` and edit our kernel lines like : 

> /kernel-6.6.15-NOUPOUE root=/dev/sda5 console=tty0 console=ttyS1 rootfstype=ext4 rw

> /kernel-6.6.15-NOUPOUE-DOCKER root=/dev/sda5 console=tty0 console=ttyS1 rootfstype=ext4 rw

* `emerge app-containers/docker app-containers/docker-cli app-containers/docker-compose`

* `systemctl enable docker`

* `reboot`

## Try docker on our kernel

Now, boot on your personal kernel and try to do the following : 

* `docker search ubuntu`

If everything works, you have docker on your kernel ! Congratulation
