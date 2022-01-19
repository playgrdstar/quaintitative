---
layout: default
title:  Setting Up a Data Lab Environment - Part 2
description: Setting Up a Data Lab Environment - Part 2
date:   2021-01-18 00:00:00 +0000
permalink: /docker_datalab_parttwo/
category: Data Engineering
---
## Setting Up a Data Lab Environment - Part 2 - Jupyter notebook in AWS with Docker

If it ain’t broke, should it still be fixed?

Why do we need to use Docker, when serving a Jupyter notebook on AWS ain’t that hard? I had posted some past tutorials on this on Medium previously.
- Basic Jupyter notebook on AWS [here][1]; and
- Jupyter notebook, with Nginx [here][2]

In this one, I will show how much more trivial the task can be when Docker is used. Just with two scripts that are at this [repo][3], we will be able, in 3 steps do what we did above.

First, we need to set up an EC2 instance at AWS and SSH in. If you need details on how to do this, look [here][4].

Once you are in, 4 lines of code will get you a Jupyter notebook up on AWS.

Clone my Github repo.
```
git clone https://github.com/playgrdstar/docker_jupyter.git
```
Get into the folder.
```
cd docker_jupyter
```
Run a bash script to install Docker and setup the necessary folders within.
```
bash initialize.sh
```
Use docker-compose to setup a container with a Jupyter image, and run it.
```
docker-compose up -d
```
That’s it! Now, when you go to \<ip\>:8888, you will see the Jupyter notebook page.

We now just need the token to get in. All you need to do is to see the containers that are running, and get the name of the container.
```
docker-compose ps
```
Access the logs with the name of the container.
```
docker logs <container_name>
```

Now just copy and paste the token password in.

To stop everything, just do this.
```
docker-compose down
```

That’s it. Easy peasy. 

There is, of course, the matter of explaining the code in _initialize.sh_ and _docker-compose.yml_, which I will go through in a subsequent post.

[1]:	https://medium.com/quaintitative/jupyter-notebook-in-the-cloud-70cbf9c2cd92
[2]:	https://medium.com/quaintitative/jupyter-notebook-on-aws-with-nginx-4326e2122096
[3]:	https://github.com/playgrdstar/docker_jupyter
[4]:	https://medium.com/quaintitative/using-amazon-web-services-ec2-for-the-first-time-100a662ab51b22