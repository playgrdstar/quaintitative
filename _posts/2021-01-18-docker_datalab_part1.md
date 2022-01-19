---
layout: default
title:  Setting Up a Data Lab Environment - Part 1
description: Setting Up a Data Lab Environment - Part 1
date:   2021-01-18 00:00:00 +0000
permalink: /docker_datalab_partone/
category: Data Engineering
---
## Setting Up a Data Lab Environment - Part 1 - I ❤️ Docker 
The reason I started using Docker was because I was trying to run Jupyter notebooks on Amazon Web Services (AWS). As that’s still something useful to know if all you want is something lightweight, and without the added steps of needing to install Docker, I’ve covered that in 2 previous posts:
- Basic Jupyter notebook on AWS [here][1]; and
- Jupyter notebook, with Nginx [here][2]

It’s a lot easier to do these. But to just use Docker for the two tasks above is frankly a waste. 

So we what we will do in this series is to get a -
- General understanding of Docker
- Set up a Jupyter notebook in AWS with Docker
- Add Flask, Nginx and WSGI to the mix
- Throw in Mongo and Postgres databases as well
- Set up a front-end to visualise our work

**Docker**
So let’s go through some basics in Docker first. 

I won’t go through the full extent of what it can do or how to install Docker (just Google!), but will just run through 2-3 key commands that will help get you an understanding of what Docker does, and get you through the series.

What Docker does is quite simple. It creates a container with a barebones OS, and adds layers of software on top, depending on what you need.

Each combination of OS and software is called an image. You can use images built by others, or roll your own. Or roll your own starting with an image built by someone else. 

So once you have an image, all you need to do is first get the image by pulling it.
```
docker pull jupyter/minimal-notebook
```
Then run it. The -p flag is essential to link port 8888 on your own computer to the port 8888 in the Docker container. 
```
docker run -p 8888:8888 jupyter/minimal-notebook
```
When you do this, the terminal will spurt out a series of logs, and you will not be able to do anything further in the terminal. To have Docker run in a detached mode, which will allow you to continue on in the terminal, just add a -d flag.
```
docker run -p -d 8888:8888 jupyter/minimal-notebook
```
Now you have an instance of a jupyter notebook running in a container, which you can access the usual way - localhost:8888.

Usually what we would do would be to install a base installation of Anaconda, and start adding on other libraries and packages as and when needed. 

With Docker, we don’t really have to do this. There are pre-defined images available that you can use, as seen in the image below which I took from [here][3]. 

For more of the most common docker commands, I find this to be a good [cheat sheet][4].


[1]:	https://medium.com/quaintitative/jupyter-notebook-in-the-cloud-70cbf9c2cd92
[2]:	https://medium.com/quaintitative/jupyter-notebook-on-aws-with-nginx-4326e2122096
[3]:	https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html#jupyter-datascience-notebook
[4]:	https://www.docker.com/sites/default/files/Docker_CheatSheet_08.09.2016_0.pdf