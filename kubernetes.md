# Kubernetes

We will orchestrate our Docker app from the previous step with Kubernetes

## Commands

First, we need to set the VM in a bridged network :
`VM > Settings > Add > Network Adaptater > custom > vmnet0`

then : `Edit > Virtual Network Editor > vmnet0 > set the right card`

Take care to keep the NAT connected to !

Now we'll put a static ip for our physical machine via the GUI ! 

Desactivate the DHCP and set :

* `192.168.2.1/24`
* `default via 192.168.2.254`

Now, restart the VM and verify that we have an `enp2s5` card in `ip addr`

Finally, try to ping the master nodes :

* `ping 192.168.2.202`
* `ping 192.168.2.210`

Our environment is working, we can advance !
