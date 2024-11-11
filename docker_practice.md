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
>LABEL image created by NOUPOUE
>RUN apk add —no-cache nano

* `docker build -t myimage-noupoue:1.0 .`
* `docker run --name MyContainer --rm -it myimage-noupoue:1.0 /bin/ash` rm option delete the containers when it stopped

* `ping cnoupoue.live` to check that your container can connect to the internet
* `uname -a` to check that the kernel version used is your personal kernel
  
Exit the container and restart it, but this time with the -d option

* `docker run --name MyContainer --rm -it -d myimage-noupoue:1.0 /bin/ash`
* `docker exec -it MyContainer /bin/ash` to connect to your container
