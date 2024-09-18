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

