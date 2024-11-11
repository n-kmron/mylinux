# Docker installation

In this file, we'll see how to install docker on our gentoo distribution to use it on our personal kernel after !

First, we have to configure our kernel with namespaces, cgroups, etc

WARNING: we now working on our Gentoo kernel to edit our personal one !

## Let's change the kernel configuration

* `ln -s /usr/src/linux-6.6.15 /usr/src/linux` to create a symbolic link and recompile our kernel

* `cd /usr/src/linux`

* `make menuconfig` and add these options (no modules (M), only \*) 

> General setup > POSIX Message Queues

> General setup > BRF subsystem > Enable bpf() system call

> General setup > Control Group support > Memory controller + IO controller + PIDs controller + Freezer controller + HugeTLB controller + Cpuset controller + Include legacy /proc/<pid>/cpuset file + Device controller + Simple CPU accounting controller + Perf controller + Support for eBPF programs attached to cgroups

> General setup > Control Group support > CPU controller > Group scheduling for SCHED_OTHER + CPU bandwidth provisionning for FAIR_GROUP_SCHED + Group scheduling for SCHED_RR/FIFO

> 

>

>

>

>

>
