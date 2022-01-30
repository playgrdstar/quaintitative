---
layout: default
title:  Introduction to Machine Learning Right Inside the Browser
description: Introduction to Machine Learning Right Inside the Browser
date:   2021-01-30 00:00:00 +0000
permalink: /ml_browser/
category: Machine Learning
---
## Introduction to Machine Learning Right Inside the Browser

I got a little bored of playing around with Tensorflow and Keras in Python, and so decided to try to apply machine learning using tensorflow.js right in the browser. Turned out to be a little more tricky than I expected, mostly because of the need to adapt to the asynchronous nature of Javascript when loading data. But it was fun.

The data and optimisation process, visualised in D3 (my own little TensorBoard, yay) can be seen [here][1].  

This is not a D3 animation, we are actually training a model right in the browser on your computer! We are training a simple linear regression model, using batch gradient descent, with the adam optimizer, and mean squared error as the loss function.

In the file data.js, we are -
- Loading the iris dataset remotely(from a gist)
- Breaking them into pre-defined batch sizes

And in the file regression2.js, we are -
- Setting up the necessary tensor flow variables and constants
- Defining a predict and a loss function
- Training a linear regression model using the loaded data, and the loss/predict functions
- Plotting a scatter plot of the iris dataset
- Plotting the training loss as the model gets trained, right inside your browser!
- Once training is completed, make a series of predictions, and use these points to plot a regression line.

The code is pretty long, and I’ve added a fair number of comments directly in the code to guide the reader along, so I will not go through it step by step.

Let me just highlight two key code blocks.

The first is the code used in data.js to load data remotely from a gist.
```
async function fetchData(){
    let response = await fetch(datasource);
    let text = await response.text();

    // Parse the text data into json with d3's function
    let data = await d3.csvParse(text);

    let categoricalTargets = ['setosa', 'versicolor', 'virginica'];

    await data.forEach(d=>{

        // Or do it with a separate categoricalTargets lookup
        categoricalTargets.forEach((c,i)=>{
            if (d.species==c){d.speciesNum=i};
        })
        x.push([+d.sepal_length]);
        y.push(+d.speciesNum/1.0);

    });

    return {data:data, x:x, y:y};
}
```	
 
We use async and await here for a very simple reason. It allows us to use the new Promise functionality in Javascript ES6. What is happening here is that each line here with an await prepended will actually wait for the line above it to complete running before it starts. 

This basically replaces the `function1().then(function2())` type syntax we are used to, and is much more easier to read and write.

The second block is the main part of the code that we use to train the linear regression model.
```
// Setting up the variables for machine learning
const learning_rate = 0.05;
const batch_size = 25;

const A = tf.variable(tf.randomNormal(shape=[1,1]));
const b = tf.variable(tf.randomNormal(shape=[1,1]));

// Alternative way of initialising the two variables
// const A = tf.variable(tf.scalar(0.1));
// const b = tf.variable(tf.scalar(0.1));

const optimizer = tf.train.adam(learning_rate);

let lossArray = [];

function predict(xs){
    return tf.tidy(()=>{
        const ys = xs.mul(A).add(b);
        return ys;
    });
}

function loss(predict, actual){
    return tf.tidy(()=>{
        return predict.sub(tf.cast(actual, 'float32')).square().mean();
    });
}

async function train(numIterations, done){
    
    const d = await fetchData();

    for (let i=0; i<numIterations; i++){
        let cost;
        const [xs,ys] = await batchData(d.x,d.y,50);
        cost = tf.tidy(()=>{
            cost = optimizer.minimize(()=>{
                const pred = predict(xs);
                const predLoss = loss(pred, ys);

                return predLoss;
                
            }, true);
            return cost;
        })

        cost.data().then((data)=>lossArray.push({i:i, error:data[0]}));
        
        if (i%100==0){
            ploterrors(lossArray.slice(i-100,i));
            await cost.data().then((data)=>console.log(i,data));
        }

        await tf.nextFrame();
    }
    done();
    // If we want to check the values of A and b
    // await A.data().then((data)=>console.log(data[0]));
    // await b.data().then((data)=>console.log(data[0]));
}
```

If you are familiar with Tensorflow in Python, this should be fairly easy to understand. If not, do take a look at the Tensorflow documentation. The key difference here is that we don’t use Sessions, but rather declare predict, loss etc as functions, that are then called later in train with the data to be used for training.

We also add `await tf.nextFrame();` here. What this does is basically make sure that the browser screen continues to render even as training goes on.   

It’s also important to understand what `tf.tidy()` is for. Whenever we use a tensor, it takes up space in our GPU. We can call `dispose` after each step to clear the space, but that would be really tedious. What `tf.tidy()` does is that it helps us clear up all the intermediate tensors.

There’s also something that puzzles me. In tensorflow.js, we can do operations either by `tf.add(a,b)` or `a.add(b)`. But somehow, only the latter works within the `optimizer.minimize` function. If anyone knows why, let me know?

The full code is [here][2].

Subscribe to more of such posts [here][3]


[1]:	https://playgrdstar.github.io/tensorflow_browser_d3/
[2]:	https://github.com/playgrdstar/tensorflow_browser_d3
[3]:	https://www.subscribepage.com/f5e3a3