# MyLinux
The goal of this work is to build your own Linux in a VM with the Gentoo Distribution

The major part of this work is related to the [Gentoo Handbook (AMD64)](https://wiki.gentoo.org/wiki/Handbook:AMD64)

## Prerequisites
* Download VMware Workstation 
* Download [Gentoo distribution](https://www.gentoo.org) > Downloads > amd64 > minimal installation cd & stage3 systemd

### VMware Workstation
Launch VMware Workstation and create a new VM

* Select 'custom' > Workstation > Install later > Select 'Linux 6.x kernel 64 bit'
* Put a name to your VM
* Select 2 processors (1 core per processor)
* 4GB for RAM 
* Use NAT
* LSI Logic
* SCSI
* Create a new virtual disk and split into multiple files
* Click on 'customize hardware' > New CD/DVD > Use ISO > Select `minimal installation cd` and check `connect at power on`
* Create the VM

Now, we have to put the VM on UEFI instead of BIOS : VM (toolbar) > Settings > Option > Advanced > UEFI

You're ready to launch the VM !

## Tutorial

Follow the `instructions.md` file to start the tutorial with a setup for gentoo (gentoo will serve as a basis to have a minimal environment in order to build our own OS from sources).

When `instructions.md` is ok, you can follow `mykernel.md` to start creating the OS

## Author

* Cameron Noupoue

## Credits 

Project devised and created during my studies at the Haute-Ecole de la Province de Liège, Belgium.
