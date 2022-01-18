---
layout: default
title:  (Almost) All the Most Common Machine Learning Algorithms in Javascript
description: (Almost) All the Most Common Machine Learning Algorithms in Javascript
date:   2021-01-17 00:00:00 +0000
permalink: /common_ml_javascript/
category: Machine Learning
---
## (Almost) All the Most Common Machine Learning Algorithms in Javascript

I did a Masters in Knowledge Engineering almost a decade ago. Back then, data science wasn’t a common term (if it was even used at all). Knowledge engineering was kind of the pre-cursor to data science today. The aim then was to use build systems that could facilitate knowledge management (the hot thing then). We learned things like rule based learning and neural networks.

My recollection of what I learnt then is pretty foggy. 

But I do remember learning that the best way to understand how a model works is to build a simple implementation of it from scratch. With the wealth of libraries we have access to today, we sometimes forget this.

So I thought it would be good to just write down a whole suite of the most common machine learning methods from scratch in Javascript, let them run right on the client-side in the browser, and visualise their results in the browser using D3.js. I managed to cover - 
- linear regression
- linear regression with stochastic gradient descent
- logistic regression with stochastic gradient descent
- linear discriminant analysis
- CART
- naive bayes
- KNN classifier
- support vector machine
- bagging
- boosting

The full code is available [here][1]. Each method is in its own .js file in the scripts folder. 

A simulation of all these methods is available [here][2]. As the data used is generated randomly and a number of the methods are stochastic in nature, you should see different results each time you refresh the browser. You can also see all the different results being printed out in the browser console.

[1]:	https://github.com/playgrdstar/machine_learning_methods_collection_javascript
[2]:	https://playgrdstar.github.io/machine_learning_methods_collection_javascript/