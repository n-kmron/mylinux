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


