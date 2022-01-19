---
layout: default
title:  Setting Up a Data Lab Environment - Part 4
description: Setting Up a Data Lab Environment - Part 4
date:   2021-01-18 00:00:00 +0000
permalink: /docker_datalab_partfour/
category: Data Engineering
---
## Setting Up a Data Lab Environment - Part 4 - Dockerfile

Sometimes, you may need more packages than what are in the images that you can pull straight from Docker Hub. 

Instead of just pulling an image, you can roll your own.

It’s pretty straightforward. In the same folder as the docker-compose.yml file, create a new folder ‘docker’, and within it, a ‘jupyter’ folder.

Inside that folder, create a ‘Dockerfile’. In the ‘Dockerfile’, first state the base image you want to build off.
```
FROM jupyter/tensorflow-notebook 
```
Set the user to be root so you are able to install with root permissions.
```	
USER root
```

Next, just prepend RUN to commands that you would usually run in a terminal to install new packages.
```
RUN pip install --no-cache-dir lxml
RUN conda install --yes --name root scrapy
RUN conda install --yes --name root spacy
RUN conda install --yes --name root gensim
RUN conda install --yes --name root nltk
RUN conda install --yes --name root pymongo
RUN conda install --yes --name root psycopg2
RUN pip install tweepy
RUN pip install awesome-slugify
RUN pip install feedparser
RUN pip install jieba
RUN mkdir .jupyter
```

Now, go into your docker-compose.yml. Replace `image: jupyter/tensorflow-notebook` with `build: docker/jupyter`.

That’s it. Now when you do a `docker-compose up -d`, you will get a Jupyter environment that has the packages you included in the Dockerfile.
