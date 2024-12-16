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
* `kubeadm token create --print-join-command` to get a command to inject in our nodes to make them join the master node (keep this command)

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

> /mnt/mongo-noupoue 192.168.2.0/24(rw,sync,no_subtree_check,no_root_squash)

* `sudo chmod 644 /etc/exports`
* `sudo exportfs -a`
* `sudo systemctl restart nfs-kernel-server` to restart the NFS server

On the gentoo, we need to copy the data from the docker volume to the NFS server :

* `docker inspect multicontainerwebapp_mongo_data` to get the path of the volume
* `scp /var/lib/docker/volumes/multicontainerwebapp_mongo_data/_data/* admin@192.168.2.202:/mnt/mongo-noupoue`
* `scp /var/lib/docker/volumes/multicontainerwebapp_mongo_data/_data/diagnostic.data/* admin@192.168.2.202:/mnt/mongo-noupoue`
* `scp /var/lib/docker/volumes/multicontainerwebapp_mongo_data/_data/journal/* admin@192.168.2.202:/mnt/mongo-noupoue`

Go back to the master node :

* `cd /home/user`
* `nano nfs-pv.yaml` and paste the code you can find [here](https://github.com/kubernetes/examples/blob/master/staging/volumes/nfs/nfs-pv.yaml)
We just need to change the `nfs-server` and replace by our NFS server `192.168.2.202`, storage with `10Mi` and `path` with `/mnt/mongo-noupoue`

* `kubectl apply -f nfs-pv.yaml`
* `kubectl get pv` to verify we have been create our NFS

The PV (persistent volume) is a storage resource independent of the pod lifecycle to survive the deletion or re-registration of pods. It also centralise the management for the admin

Return back to the NFS server (192.168.2.202)

* `sudo docker run -d -p 5000:5000 --restart=always --name registry registry:2`

Return back to the Gentoo with Docker and let's export our images

WARNING: to get the 4.4 version for mongodb, you need to rebuild your image and Gentoo after modify the `Dockerfile`

> FROM mongo:4.4
>
> LABEL maintainer="Cameron Noupoue"
>
> EXPOSE 27017

* `docker build -t databasenoupoue:k8 ./databaseNOUPOUE`

WARNING: we also need to edit the `nginx.conf` in the frontend to regulate the k8s format (backendNOUPOUE is not good, we need lowercase)

* `nano ./frontendNOUPOUE/nginx.conf` -> change `backendNOUPOUE` in `backendnoupoue-service`

We can build a new image to not affect our actual config for Gentoo 

* `docker build -t frontendnoupoue:k8 ./frontendNOUPOUE`

Now, we can export our images

* `docker save -o frontend.tar frontendk8noupoue` 
* `docker save -o backend.tar backendnoupoue` 
* `docker save -o database.tar databasenoupoue`
* `docker save -o oauth.tar oauth2noupoue`
* `scp frontend.tar backend.tar database.tar oauth.tar admin@192.168.2.202:/mnt/mongo-noupoue`

Now, we can delete the `.tar` files

Go back to the NFS server (192.168.2.202)

* `sudo docker load -i /mnt/mongo-noupoue/frontend.tar`
* `sudo docker load -i /mnt/mongo-noupoue/backend.tar`
* `sudo docker load -i /mnt/mongo-noupoue/database.tar`
* `sudo docker load -i /mnt/mongo-noupoue/oauth.tar`

We'll add tag before pushing our images to the local registry 

* `sudo docker tag frontendk8noupoue:1.0 localhost:5000/frontendnoupoue`
* `rm frontend.tar`
* `sudo docker tag backendnoupoue:1.0 localhost:5000/backendnoupoue`
* `rm backend.tar`
* `sudo docker tag databasenoupoue:1.0 localhost:5000/databasenoupoue`
* `rm database.tar`
* `sudo docker tag quay.io/oauth2-proxy/oauth2-proxy localhost:5000/oauth2noupoue`
* `rm oauth.tar`

We are now able to push our images to the local registry 

* `sudo docker push localhost:5000/frontendnoupoue`
* `sudo docker push localhost:5000/backendnoupoue`
* `sudo docker push localhost:5000/databasenoupoue`
* `sudo docker push localhost:5000/oauth2noupoue`

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

Now, we will write a `.yaml` file to specify our deployment process (to access our images in the private registry)

On the master node : 

* `ssh -v user@192.168.2.210`
* `cd /home/user`

We need now to do a secret to encrypt the password with k8s

* `kubectl create secret generic secretpassword --from-literal=password=password`

Let's create the `.yaml` deployment for the frontend

* `vim frontend-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontendnoupoue
  labels:
    app: frontendnoupoue
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontendnoupoue
  template:
    metadata:
      labels:
        app: frontendnoupoue
    spec:
      containers:
      - name: frontendnoupoue
        image: 192.168.2.202:5000/frontendnoupoue
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: frontendnoupoue-service
spec:
  selector:
    app: frontendnoupoue
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

Let's create the `.yaml` deployment for the backend

* `vim backend-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backendnoupoue
  labels:
    app: backendnoupoue
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backendnoupoue
  template:
    metadata:
      labels:
        app: backendnoupoue
    spec:
      containers:
        - name: backendnoupoue
          image: 192.168.2.202:5000/backendnoupoue
          ports:
            - containerPort: 3000
          env:
            - name: MONGO_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: secretpassword
                  key: password
            - name: MONGO_URI
              value: mongodb://cameron:$(MONGO_DB_PASSWORD)@databasenoupoue-service:27017/
---
apiVersion: v1
kind: Service
metadata:
  name: backendnoupoue-service
spec:
  selector:
    app: backendnoupoue
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
  type: ClusterIP
```

Let's create the `.yaml` deployment for the database

* `vim mongo-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongonoupoue
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongonoupoue
  template:
    metadata:
      labels:
        app: mongonoupoue
    spec:
      containers:
        - name: mongonoupoue
          image: 192.168.2.202:5000/databasenoupoue
          ports:
            - containerPort: 27017
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              value: "cameron"
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: secretpassword
                  key: password
            - name: MONGO_URI
              value: mongodb://cameron:$(MONGO_INITDB_ROOT_PASSWORD)@databasenoupoue-service:27017/
            - name: MONGO_INITDB_DATABASE
              value: chat
          volumeMounts:
            - name: mongonoupoue-storage
              mountPath: /data/db
      volumes:
        - name: mongonoupoue-storage
          nfs:
            server: 192.168.2.202
            path: /mnt/mongo-noupoue
---
apiVersion: v1
kind: Service
metadata:
  name: databasenoupoue-service
spec:
  ports:
    - port: 27017
  selector:
    app: mongonoupoue
```

Let's create the `.yaml` deployment for the PVC (the PVC is the PV claim, it ask the PV to consume the data in the peristent volume). Be careful, the data's quantity must be lower or equal to the PV

* `vim nfs-pvc.yaml`
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Mi
```

Let's create the `.yaml` deloyment for the oauth

* `vim oauth-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oauth2noupoue
spec:
  replicas: 1
  selector:
    matchLabels:
      app: oauth2noupoue
  template:
    metadata:
      labels:
        app: oauth2noupoue
    spec:
      containers:
        - name: oauth2noupoue
          image: 192.168.2.202:5000/oauth2noupoue
          ports:
            - containerPort: 4180
          env:
            - name: OAUTH2_PROXY_HTTP_ADDRESS
              value: "0.0.0.0:4180"
            - name: OAUTH2_PROXY_PROVIDER
              value: "gitlab"
            - name: OAUTH2_PROXY_OIDC_ISSUER_URL
              value: "http://dummy.com/"
            - name: OAUTH2_PROXY_OIDC_JWKS_URL
              value: "http://dummy.com/"
            - name: OAUTH2_PROXY_CLIENT_ID
              value: "client_id"
            - name: OAUTH2_PROXY_CLIENT_SECRET
              value: "client_secret"
            - name: OAUTH2_PROXY_REDIRECT_URL
              value: "http://192.168.2.210:4180/oauth2/callback"
            - name: OAUTH2_PROXY_UPSTREAMS
              value: "http://frontendnoupoue-service/"
            - name: OAUTH2_PROXY_SSL_INSECURE_SKIP_VERIFY
              value: "true"
            - name: OAUTH2_PROXY_COOKIE_SECRET
              value: "12345678901234567890123456789000"
            - name: OAUTH2_PROXY_COOKIE_SECURE
              value: "false"
            - name: OAUTH2_PROXY_COOKIE_PATH
              value: "/"
            - name: OAUTH2_PROXY_HTPASSWD_FILE
              value: "/etc/oauth2-proxy/.htpasswd"
            - name: OAUTH2_PROXY_SKIP_OIDC_DISCOVERY
              value: "true"
---
apiVersion: v1
kind: Service
metadata:
  name: oauth2noupoue-service
spec:
  ports:
    - port: 80
      targetPort: 4180
      nodePort: 30000
  selector:
    app: oauth2noupoue
  type: NodePort
```

Now we need to apply the `.yaml` deployments

* `kubectl apply -f frontend-deployment.yaml`
* `kubectl apply -f backend-deployment.yaml`
* `kubectl apply -f mongo-deployment.yaml'
* `kubectl apply -f oauth-deployment.yaml`
* `kubectl apply -f nfs-pvc.yaml'

If you have issues and you need to update your `.yaml` deployment

* `kubectl delete -f <your-yaml.yaml>`

Because, if you delete only the pod, the master node will automaticly restart another pod because it is in the deployment

* `kubectl get pods` to see the results
* `kubectl logs <pod-name>` to see the logs
* `kubectl describe pvc mongo-pvc` to see if the PVC works (the status should be 'BOUND')
* `kubectl get services` to see the services
Everything should be as <RUNNING>

To access our app, we need to go to the master node IP : 192.168.2.210:<port>. We have the port by doing `kubectl get pods`

If you have storage issues regarding mongo, you should delete files in `/mnt/mongo-noupoue` in the `192.168.2.202` VM

I added a script to undeploy each script and another to deploy everything, when we do major changes (like add the secretkey password, ...) it does not work if we don't undeploy then deploy everything.



* `chmod +x undeploy.sh`
* `chmod +x deploy.sh`

Last tip : if it still does not work because of `websocket:30000 unavailable` we need to remove the node port in the `oauth2noupoue-service`. So, when we'll launch the app, we will need to check `kubectl get svc` external port for oauth2

Each service has only the internal port that is open to only allow clusters to access them. Apart from the service `oauth` who's accessible from an external port (30000) because it's the entry point of our app.
