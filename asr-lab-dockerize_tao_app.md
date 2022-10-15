## Dockerizing TAO web application
------------------------------------

**goals:**
 * Dockerize the TAO web application from the freely available source code (community edition)
 * Deploy the application on a single host using `docker-compose` tool
 * Deploy the application on a k8s cluster using a cloud kubernetes service (Azure?)

Setup:
======

**tools:** 
**requirements:** have an installed docker engine and docker cli on a Linux host.

Outline (to be removed at the end)
- describe the application, the goal
- understand the application architecture and requirements from hosting environment (volumes, networking setup, proxy)
- select docker base images and their requirements
- dockerizing the web application
- prepare deployment for a single node (docker compose)
- switch to kubernetes

# An overview of the application
 * TAO is a web application used for managing online assesments for students, with features like grading, scheduled tests, groups, questions shuffling, etc.
 * We'll be using the community edition, which is freely available.
 * The application is made up of three components:
 	- apache2 web server
 	- php engine
 	- db server

# Installation summary
The installation guide is available [https://www.taotesting.com/user-guide/installation-and-upgrade/ubuntu-and-debian/](https://www.taotesting.com/user-guide/installation-and-upgrade/ubuntu-and-debian/). Here are the steps summarized:
 * The first step is to set up the web server, install php engine (we do not care if a recent version is used instead of the 7.2), and the needed php libs, enable php modules and set up a virtual host.
 * Next, we have to setup a db server, create the database for tao, the main role, and assign privileges. All those steps are normally done with the master database account
 * Then, we have to host the TAO code in the Apache2 server and run the installation script, which will create tables, and other objects (it'll use the database and the user created beforehand)

# Guidelines and suggestions for installing on a Docker engine
 
 * Obviously, for deploying the app on a docker environment or any other containers orchestration environment, we have to dockerize the web app part first. Docker images for the MySQL or PostgreSQL are readily available on the docker hub (official images), and they only need setup like adding storage and networking.
 
 * To create the `tao-webapp` image (that's the name we will use for our image), we have to select a suitable *base image* and add layers on it using a custom dockerfile (in order to make customizations and add code.) (There are other options like creating images from a live filesystrem or working inside the base container.).
 Since we need php and apache, we have to use the `php:apache` image or preferably `php:apache-buster` (because I'm not sure whether `php:apache` is built on top of debian, so that it offers the `apt` tool we may need later).

 We have also to add to the image the code, and run the script for configuring the tao database (the toa db it self must be created before hand by the db container).
 
 * I prefer you use PostgreSQL as a database server instead of MySQL. Look at the configuration of the official image (https://hub.docker.com/_/postgres) and particularly to the following attributes:
 	* POSTGRES_PASSWORD (env variable)
 	* PGDATA (env variable) or the pgdata (volume)
 	* Initialization scripts

 	You may also want to look at a different way to pass the POSTGRES_PASSWORD secret to the container, in the Secrets section.

	The PostgreSQL server have a way to run script only once, typically for setting up users/roles/.. and creating databases. We will have to do this on our database container before installing the TAO app. For this purpose, we have to create a bash script containing the instructions.

	You will also need to attach a volume to the container, where the db container will store the data. Using an external volume and mounting it to the `/var/lib/postgresql/data` ensure that the data persists across many containers instances or reboots.