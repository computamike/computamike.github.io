---
title: Building on WordPress
date: 2020-01-29 12:12:00 Z
tags:
- Development
---

A project I am working on is related to the REST API for WooCommerce.  This has meant that for development purposes I need to stand up a WooCommerce instance that I can interact with.

We've recently started getting into docker more and more, so I thought I'd try my hand at building a docker image.  I borrowed quite heavily from the example docker-compose.yml file and the documentation from the [official WordPress docker image](https://hub.docker.com/_/wordpress).

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

## Log tracing
I started looking in desparation in the WordPress container logs - and to my surprise I found this right at the top of the wordpress logs.

```
WordPress not found in /var/www/html - copying now...
WARNING: /var/www/html is not empty! (copying anyhow)
Complete! WordPress has been successfully copied to /var/www/html
...
...

```

It seems that the WordPress image is using the entrypoint to copy the Wordpress System onto the container.  If you search google for the phrase `Complete! WordPress has been successfully copied to` it brings you to this eventual file : [https://github.com/docker-library/wordpress/blob/master/docker-entrypoint.sh](https://github.com/docker-library/wordpress/blob/master/docker-entrypoint.sh)

which appears to be the entrypoint script for the wordpress container.  This script copies the wordpress files from /usr/src into /var/www/html and then creates a .htaccess file for that folder.

Docker has the following guidelines for Best Practice :

https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#entrypoint

> The best use for ENTRYPOINT is to set the image’s main command, allowing that image to be run as though it was that command (and then use CMD as the default flags).

I'm of the opinion that the existing docker file causes a problem for developers looking to build on top of WordPress, and that the entrypoint for a container shouldn't be used to install the code, the docker file is the place to do that.

Anyone got any ideas ? 



 
