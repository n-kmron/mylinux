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

Now, we need to connect to the master node to register worker nodes in the cluster: 

* `ssh -v user@192.168.2.210` (P@ssw0rd)
* `kubectl get pods --all-namespaces` to verify if the master node is ready to get some worker nodes. We should see all serivces in a RUNNING status
* `kubeadm token create - -print-join-command` to get a command to inject in our nodes to make them join the master node (keep this command)

Connect with ssh to the master nodes (.211 et .212) and paste the command you got with a `sudo` prefix

* `ssh -v user@192.168.2.<worker-node>` (P@ssw0rd)
* sudo + paste the command

Get back to the master node :

* `kubectl get nodes` to verify that our worker nodes are here

We just need to wait for the workers are in a READY status

Now, we'll configure the NFS server to have a remote storage for our cluster

* `ssh -v admin@192.168.2.202`
* `sudo mkdir -p /mnt/mongo-noupoue`
* `sudo chmod 777 /mnt/mongo-noupoue`
* `sudo chmod 777 /etc/exports` (P@ssw0rd)
* `nano /etc/exports`

> /mnt/noupoue 192.168.2.0/24(rw,sync,no_subtree_check,no_root_squash)

* `sudo chmod 644 /etc/exports`
* `sudo exportfs -a`
* `sudo systemctl restart nfs-kernel-server` to restart the NFS server

On the gentoo, we need to copy the data from the docker volume to the NFS server :

* `docker inspect multicontainerwebapp_mongo_data` to get the path of the volume
* `scp /var/lib/docker/volumes/multicontainerwebapp_mongo_data/_data/* admin@192.168.2.202:/mnt/mongo-noupoue`
* `scp /var/lib/docker/volumes/multicontainerwebapp_mongo_data/_data/diagnostic.data/* admin@192.168.2.202:/mnt/mongo-noupoue`
* `scp /var/lib/docker/volumes/multicontainerwebapp_mongo_data/_data/journal/* admin@192.168.2.202:/mnt/mongo-noupoue`

Return back to the NFS server (192.168.2.202)

* `sudo docker run -d -p 5000:5000 --restart=always --name registry registry:2`

Return back to the Gentoo with Docker and let's export our images

* `docker save -o frontend.tar frontendnoupoue` 
* `docker save -o backend.tar backendnoupoue` 
* `docker save -o database.tar databasenoupoue`
* `scp frontend.tar backend.tar database.tar admin@192.168.2.202:/mnt/mongo-noupoue`

Now, we can delete the `.tar` files

Go back to the NFS server (192.168.2.202)

* `sudo docker load -i /mnt/mongo-noupoue/frontend.tar`
* `sudo docker load -i /mnt/mongo-noupoue/backend.tar`
* `sudo docker load -i /mnt/mongo-noupoue/database.tar`

We'll add tag before pushing our images to the local registry 

* `sudo docker tag frontendnoupoue localhost:5000/frontendnoupoue`
* `rm frontend.tar`
* `sudo docker tag backendnoupoue localhost:5000/backendnoupoue`
* `rm backend.tar`
* `sudo docker tag databasenoupoue localhost:5000/databasenoupoue`
* `rm database.tar`

We are now able to push our images to the local registry 

* `sudo docker push localhost:5000/frontendnoupoue`
* `sudo docker push localhost:5000/backendnoupoue`
* `sudo docker push localhost:5000/databasenoupoue`

Now, we need to make the registry accessible from each nodes (.210, .211, .212) 

* `sudo nano /etc/containerd/config.toml`

And add this at the end :

```toml
[plugins."io.containerd.grpc.v1.cri".registry.mirrors."192.168.2.202:5000"]
  endpoint = ["http://192.168.2.202:5000"]
[plugins."io.containerd.grpc.v1.cri".registry.insecure]
  endpoint = ["http://192.168.2.202:5000"]
```

* `sudo systemctl restart containerd`
