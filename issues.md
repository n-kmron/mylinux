# My issues

## This file reports the issues I encountered during my kernel's journey: 

* During the step 2, I was not able to connect on remote using SSH while the SSH was activated
  > the solution was to create a new user and add it the wheel group

* During the step 2, my /dev/sda5 type was not ext4 so I was not able to mount it 

* During the first step, I had issues in my kernel configuration (make menuconfig) so when I changed some things, I forgot to copy the bzImage into /boot so when I booted my kernel, the changes didn't appear.

* During the step 2, /dev/sda1 and /dev/sda2 were not mounted on /efi and /boot so, my grub was not up to date.

* At the end of step 2, when I reboot on my new kernel, my keyboard was still qwerty, because the `/etc/rc` script was not correctly executed, so I had to reput the commands to make everything good.
