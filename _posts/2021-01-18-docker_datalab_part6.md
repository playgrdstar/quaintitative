---
layout: default
title:  Setting Up a Data Lab Environment - Part 6
description: Setting Up a Data Lab Environment - Part 6
date:   2021-01-18 00:00:00 +0000
permalink: /docker_datalab_partsix/
category: Data Engineering
---
## Setting Up a Data Lab Environment - Part 6 - Serve a Flask

Adding Flask to the mix isn’t strictly required for this. But I think it’s great to be able to 
- do the analysis;
- save the data to either the Postgres or Mongo database; and 
- serve the analysis from a Flask server 

 I explained how to set up a Flask server in a previous [post][1]. Here, we show how to use Docker to do the same.

First, we have to add Flask to the _docker-compose.yml_ file.
```
flaskapp:
    build: docker/flask
    ports:
        - "80:80"
    volumes: 
        - ./docker/flask/app:/app
        - ./data:/app/data
    environment:
        - FLASK_APP=main.py
        - FLASK_DEBUG=1
        - 'RUN=flask run --host=0.0.0.0 --port=80'
    command: flask run --host=0.0.0.0 --port=80
```
We build an image from Dockerfile in the folder ‘docker/flask’ (which I will go through next); connect the container’s port 80 to port 80 in the outside world, map the volumes, and then set the variables and commands needed to get Flask up and running.

Next, we create a folder for _flask_ in the _docker_ folder that we had created previously. Within it, we create an _app_ folder, and a Dockerfile.

In the Dockerfile, we pull a Docker image, and then install some libraries and copy the Flask app files from the local machine to the container’s app folder.
```
FROM tiangolo/uwsgi-nginx-flask:python3.6

RUN pip install pymongo
RUN pip install psycopg2
RUN pip install tweepy

COPY ./app /app
```
And that’s it. You can just adapt the files in the app folder I provided. It goes slightly beyond what I covered on Flask previously, but I will go into more details on Flask in subsequent posts.

The files for this tutorial are available [here][2].

[1]:	https://medium.com/quaintitative/almost-the-simplest-server-ever-c74b865c2e3c
[2]:	https://github.com/playgrdstar/docker_datalab
