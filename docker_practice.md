# Docker practice 

Let's practice Docker on our kernel !

We'll do some exercices to use Docker, Dockerfile, Docker compose, etc

## 1. Introduction docker

Install a nginx container and make it accessible via port 8080 (use docker search, docker pull, docker run with the --name,-p 8080:80 and -d options)
Test that you can access the nginx homepage using your gentoo machine IP. Stop the container and delete it.

*  `docker search nginx`
*  `docker pull nginx`
*  `docker run -name mynginx -p 8080:80 -d nginx`
*  `ip addr` to get our ip
*  `http://<ip>:8080`
*  `docker stop mynginx`
*  `docker rm mynginx`

## 2. Introduction Dockerfile
