# Docker practice 

Let's practice Docker on our kernel !

We'll do some exercices to use Docker, Dockerfile, Docker compose, etc

## 1. Introduction docker

Install a nginx container and make it accessible via port 8080 (use docker search, docker pull, docker run with the --name,-p 8080:80 and -d options)
Test that you can access the nginx homepage using your gentoo machine IP. Stop the container and delete it.

*  `docker search nginx`
*  `docker pull nginx`
*  `docker run --name mynginx -p 8080:80 -d nginx`
*  `ip addr` to get our ip
*  `wget http://<ip>:8080`
*  `docker stop mynginx`
*  `docker rm mynginx`

## 2. Introduction Dockerfile

* `mkdir -p /root/FirstDockerFile`
* `cd /root/FirstDockerFile`
* `nano Dockerfile`

>FROM alpine
>
>LABEL image created by NOUPOUE
>
>RUN apk add —no-cache nano

* `docker build -t myimage-noupoue:1.0 .`
* `docker run --name MyContainer --rm -it myimage-noupoue:1.0 /bin/ash` rm option delete the containers when it stopped

* `ping cnoupoue.live` to check that your container can connect to the internet
* `uname -a` to check that the kernel version used is your personal kernel
  
Exit the container and restart it, but this time with the -d option

* `docker run --name MyContainer --rm -it -d myimage-noupoue:1.0 /bin/ash`
* `docker exec -it MyContainer /bin/ash` to connect to your container

## 3. A multi-container chat web application

We will build a multi-container chat web application using websockets. 
This application will include a frontend running on nginx, a nodejs backend and a NoSQL database.

* `mkdir -p /root/MultiContainerWebapp`
* `cd /root/MultiContainerWebapp`

On your laptop, download the provided files and send them using scp to your VM
* `useradd -m -G users,wheel cameron` to create an user for scp transfer
* `passwd cameron` (NOUPOUE2024)
* `ip addr` to see which ip to use with scp
* `scp /home/cameron/Downloads/index.html cameron@<ip>:/home/cameron`
* `scp /home/cameron/Downloads/nginx.conf cameron@<ip>:/home/cameron`
* `scp /home/cameron/Downloads/package.json cameron@<ip>:/home/cameron`
* `scp /home/cameron/Downloads/server.js cameron@<ip>:/home/cameron`
* `mv /home/cameron/* .`

* `mkdir frontendNOUPOUE backendNOUPOUE databaseNOUPOUE` create our file tree with 3 directories
* `mv index.html nginx.conf frontendNOUPOUE`
* `mv server.js package.json backendNOUPOUE`

In the `index.html` file, we need to edit the following line : 
> const ws = new WebSocket(´ws://backend:3000/ws’);

To replace by : 
> const ws = new WebSocket(´ws://\<our-ip\>:3000/ws’);


In the `nginx.conf` file, we need to edit the following line : 
> proxy_pass http://backend:3000;

To replace by : 
> proxy_pass http://backendNOUPOUE:3000;

You must write the 3 dockerfiles. The base images will be: nginx for the frontend (use
your personal version, the one you were assigned during the theoretical course!), 
Let's write 3 dockerfiles (frontend, backend and database)

* `cd frontendNOUPOUE && nano Dockerfile`

> FROM nginx:1.19.5
> 
> LABEL maintainer="Cameron Noupoue"
> 
> COPY index.html /usr/share/nginx/html/index.html
> 
> COPY nginx.conf /etc/nginx/nginx.conf
> 
> RUN apt-get update && apt-get install -y iproute2 iputils-ping
>
> EXPOSE 80

* `cd backendNOUPOUE && nano Dockerfile`

> FROM node:14
>
> LABEL maintainer="Cameron Noupoue"
>
> WORKDIR /app
>
> COPY package.json /app/
>
> RUN npm install
>
> COPY server.js /app/
>
> RUN apt-get update && apt-get install -y iproute2 iputils-ping
>
> EXPOSE 3000
>
> CMD ["node", "server.js"]

* `cd databaseNOUPOUE && nano Dockerfile`

> FROM mongo
>
> LABEL maintainer="Cameron Noupoue"
> 
> RUN apt-get update && apt-get install -y iproute2 iputils-ping
>
> EXPOSE 27017

* `docker network create —-driver bridge noupoue-network` to create a docker bridge network to be able to communicate between your
containers using their names

Let's start this application using docker run commands

* `docker build -t databasenoupoue:1.0 ./databaseNOUPOUE`
* `docker build -t backendnoupoue:1.0 ./backendNOUPOUE`
* `docker build -t frontendnoupoue:1.0 ./frontendNOUPOUE`

* `docker run --name databaseNOUPOUE --network noupoue-network -e MONGO_INITDB_ROOT_USERNAME=cameron -e MONGO_INITDB_ROOT_PASSWORD=password -d databasenoupoue:1.0`
* `docker run --name backendNOUPOUE --network noupoue-network -e MONGO_URI="mongodb://cameron:password@databaseNOUPOUE:27017" -p 3000:3000 -d backendnoupoue:1.0`
* `docker run --name frontendNOUPOUE --network noupoue-network -p 8080:80 -d frontendnoupoue:1.0`

* `docker ps` to see if everything is good
* `ip addr`

You now should be able to go on `http://\<our-ip\>:8080` and use our application

NB: storage is managed by docker in subdirectories of /var/lib/docker/volumes

## 4. Introduction à Docker compose 

* `docker stop $(docker ps -q)`
* `docker rm $(docker ps -aq)`
* `nano /root/MultiContainerProject/MyFirstDockerComposeFile.yaml`

```yaml
version: "1.0"

services:
  databasenoupoue:
    image: "databasenoupoue:1.0"
    container_name: "databaseNOUPOUE"
    environment:
      MONGO_INITDB_ROOT_USERNAME: cameron
      MONGO_INITDB_ROOT_PASSWORD: password
    volumes:
      - mongo_data:/data/db
    networks:
      - noupoue-network

  backendnoupoue:
    image: "backendnoupoue:1.0"
    container_name: "backendNOUPOUE"
    environment:
      MONGO_URI: mongodb://cameron:password@databaseNOUPOUE:27017
    networks:
      - noupoue-network
    ports:
      - "3000:3000"

  frontendnoupoue:
    image: "frontendnoupoue:1.0"
    container_name: "frontendNOUPOUE"
    networks:
      - noupoue-network
    ports:
      - "8080:80"

volumes:
  mongo_data:

networks:
  noupoue-network:
    driver: bridge
    name: noupoue-network
    external: true
```

* `docker compose -f MyFirstDockerComposeFile.yaml up -d` to launch our app

* `docker-compose -f docker-compose.yml up -d --scale backendnoupoue=2` to split the backend load into 2 containers
WARNING: to do it, we must remove the `container_name` in the yaml file and the `port 3000` too. We also need to remove in the `index.html` the :3000 port in the `const ws`.

* `tune2fs -l /dev/sda5`


### Add a container handling authentication for this application using the oauth2-proxy image.

* `emerge app-admin/apache-tools` to have apache to host oauth2 and get the `/usr/bin/htpasswd` file (serve to do our local auth)
* `mkdir -p oauth2NOUPOUE`
* `htpasswd -nbB cameron NOUPOUE2024 > ./oauth2NOUPOUE/.htpasswd` to generate our credentials

We also need to edit the `MyFirstDockerComposeFile.yaml` to add a service for oauth2:

```yaml
oauth2noupoue:
    image: quay.io/oauth2-proxy/oauth2-proxy:latest
    container_name: oauth2NOUPOUE
    ports:
     - "80:4180"
    environment:
      - OAUTH2_PROXY_HTTP_ADDRESS=0.0.0.0:4180
      - OAUTH2_PROXY_PROVIDER=gitlab
      - OAUTH2_PROXY_OIDC_ISSUER_URL=http://dummy.com/
      - OAUTH2_PROXY_OIDC_JWKS_URL=http://dummy.com/
      - OAUTH2_PROXY_CLIENT_ID=client_id
      - OAUTH2_PROXY_CLIENT_SECRET=client_secret
      - OAUTH2_PROXY_UPSTREAMS=http://frontendnoupoue:80/
      - OAUTH2_PROXY_SSL_INSECURE_SKIP_VERIFY=true
      - OAUTH2_PROXY_COOKIE_SECRET=12345678901234567890123456789000
      - OAUTH2_PROXY_COOKIE_SECURE=false
      - OAUTH2_PROXY_COOKIE_PATH=/
      - OAUTH2_PROXY_COOKIE_DOMAINS=".localhost.com"
      - OAUTH2_PROXY_HTPASSWD_FILE=/etc/oauth2-proxy/.htpasswd
      - OAUTH2_PROXY_SKIP_OIDC_DISCOVERY=true
    volumes:
      - ./oauth2NOUPOUE/.htpasswd:/etc/oauth2-proxy/.htpasswd:ro
    networks:
      - noupoue-network
```

Now, we need to delete our images and recompose 
* `docker compose -f MyFirstDockerComposeFile.yaml down`
* `docker images` to get id of images to delete
* `docker rmi <img_id>` to delete our images
* `docker build -t <imagenoupoue>:1.0 ./<pathNOUPOUE>` to rebuild our images (only frontend, backend and database because oauth will be pull)
* `docker compose -f MyFirstDockerComposeFile.yaml up -d`

Everything is good ! You can go on `<our-ip>` to see the result (you can get <our-ip> by doing `ip addr show enp2s1`)
