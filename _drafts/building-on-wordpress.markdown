---
title: Building on WordPress
date: 2020-01-29 12:12:00 Z
tags:
- Development
---

A project I am working on is related to the REST API for WooCommerce.  This has meant that for development purposes I need to stand up a WooCommerce instance that I can interact with.

We've recently started getting into docker more and more, so I thought I'd try my hand at building a docker image.  I borrowed quite heavily from the example docker-compose.yml file and the documentation from the official WordPress docker image - located here : [https://hub.docker.com/_/wordpress](https://hub.docker.com/_/wordpress)

My plan was to create a shell script that I could shell out that would configure my WordPress instance correctly.  I wanted to run this as part of the image creation, so I started experimenting with docker-compose and docker build.

As I said previously - I'm somewhat new to docker, but here's what I understand by the difference between Docker and Docker Compose.

* Docker (or docker build) let's me create an image
* Docker Compose lets me create containers and set up an environment for containers, based on images.

Docker Compose can reference an image (for example : wordpress:latest) but equally can reference the output of a build.

So what I thought of doing was creating a docker file that I could run, and base that on wordpress:latest, and then add some RUN steps that could run a shell script that would set it all up.

Except that it doesn't work.

I have a proof of concept here that should demonstrate this here : 

https://github.com/computamike/wordpress-woocommerce-docker

Clone this repo to a folder somewhere, and from that folder execute : 

`docker-compose -f "docker-compose.yml" up -d --build`

Here is my example output : 

```
docker-compose -f "docker-compose.yml" up -d --build
Building wordpress
Step 1/1 : FROM wordpress:latest
 ---> f790ed010456

Successfully built f790ed010456
Successfully tagged wordpress-woocommerce-docker_wordpress:latest
wordpress-woocommerce-docker_db_1 is up-to-date
wordpress-woocommerce-docker_wordpress_1 is up-to-date
PS C:\Users\mhingley\code\wordpress-woocommerce-docker> docker-compose -f "docker-compose.yml" up -d --build
Creating network "wordpress-woocommerce-docker_default" with the default driver
Creating volume "wordpress-woocommerce-docker_db_data" with default driver
Building wordpress
Step 1/4 : FROM wordpress:latest
 ---> f790ed010456
Step 2/4 : COPY setup-script.sh /var/www/html/setup-script.sh
 ---> 0ee70885a5d7
Step 3/4 : RUN chmod +x /var/www/html/setup-script.sh
 ---> Running in 0ab0633ca24e
Removing intermediate container 0ab0633ca24e
 ---> 2f431181f649
Step 4/4 : RUN  /var/www/html/setup-script.sh
 ---> Running in 46290d040a04
TERM environment variable not set.
WordPress Directory Contents:
=============================
total 4
-rwxr-xr-x 1 root root 113 Jan 29 12:44 setup-script.sh
Removing intermediate container 46290d040a04
 ---> 05a31f88bbaf

Successfully built 05a31f88bbaf
Successfully tagged wordpress-woocommerce-docker_wordpress:latest
Creating wordpress-woocommerce-docker_db_1 ... done
Creating wordpress-woocommerce-docker_wordpress_1 ... done
```

So the script output shows only the script being present - so where's the rest of wordpress?



