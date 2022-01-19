---
layout: default
title:  Setting Up a Data Lab Environment - Part 3
description: Setting Up a Data Lab Environment - Part 3
date:   2021-01-18 00:00:00 +0000
permalink: /docker_datalab_partthree/
category: Data Engineering
---
## Setting Up a Data Lab Environment - Part 3 - Bashing and composing

We used two scripts _initialize.sh_ and _docker-compose.yml_to help us serve Jupyter notebooks from AWS in the [part 2][1] of this series.

Now, let’s explain how these two scripts work.

**Bashing**
First, the bash script - _initialise.sh_.

Bash scripts are codes that can be run in the terminal shell in a Unix/Linux environment. _initialise.sh_ helps us install, set the necessary permissions and create some folders.

We first update the environment we are in. `sudo` is used to allow us to undertake updates/installation as an admin. We also install tree to allow us to visualise folders as a tree.
```
# Update
sudo apt-get update

# Install tree
sudo apt install tree
```

Next, we install Docker.
```
# Download and install docker-compose
sudo curl -L https://github.com/docker/compose/releases/download/1.15.0/docker-compose-`uname -s`-`uname -m` > docker-compose
sudo mv docker-compose /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
And create some directories.
```
mkdir docker
mkdir docker/jupyter
mkdir notebook
```
And restart.
```
sudo reboot
```

**Composing**

Next _docker-compose.yml._
This barely scratches the surface of what Docker can do. But it’s good to start simple.
```
version: '3'
services:
    jupyterone:
    image: jupyter/tensorflow-notebook
    ports:
        - "8888:8888"
    volumes:
        - .:/home/jovyan/work
```

What we are doing here is to ask Docker to start a container named ‘jupyterone’, using the image ‘jupyter/tensorflow-notebook’ pulled from Docker Hub. We then connect the 8888 port in the container to the AWS EC2 instance’s 8888 port. The current folder we are in is also mapped to the folder ‘/home/jovyan/work’ in the container.

And that’s it. Pretty simple isn’t it?

[1]:	https://github.com/playgrdstar/docker_jupyter