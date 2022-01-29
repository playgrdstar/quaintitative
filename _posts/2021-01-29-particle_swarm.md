---
layout: default
title:  Nature and Math - Particle Swarm Optimisation
description: Nature and Math - Particle Swarm Optimisation
date:   2021-01-29 00:00:02 +0000
permalink: /particle_swarm/
category: Generative Art
---
## Nature and Math: Particle Swarm Optimisation

I was just looking for alternate portfolio optimisation methods when I came across particle swarm optimisation.

The basic idea behind particle swarm optimisation is simple, yet intriguing. And it is pretty much an extension of the way autonomous particles are typically coded in p5.js. 

The particle swarm optimisation algorithm reminded me so much of what I coded when I ported Nature of Code’s **flocking** example to three.js (code [here][1]; demo [here][2]). So down the rabbit hole I went again. I took the particle swarm optimisation algorithm, and mashed it together with the flocking example in p5.js.

**The outcome of this experiment can be viewed [here][3].

What’s happening with this mass of particles, you might ask?

- First, we have chosen an objective function - in this case $100(y-x^2)^2 + (1-x)^2$
- Second, we set the search space - i.e. the bounds within which we search for the minimum solution for the objective function
- Then we initialise 300 particles at random positions.
- A velocity is applied to each particle’s position and updated at each iteration/frame. The velocity at each step is influenced by 
	- 1) an inertia component, which keeps it on the path it was on; 
	- 2) a cognitive component, which serves as the particle’s memory and steers it to return to regions where the better (lower) results (found by substituting the position of the particle into the objective function) were found by the particle; 
	- 3) a social component, which steers the particle towards the best region the entire swarm of particles has found so far.
- This process is repeated until the particles converge on a solution is found.

So what you are seeing is a mass of particles autonomously finding a solution to the function by themselves, over time.

As always, I’ve added extensive comments to the code that can be found [here][4].

Now, let me run through the code. 

This is the objective function we are trying to find the minimum of.
```
function f1(pos){
    // Scale the position so we can see on the screen
    var tempx = pos.x/100;
    var tempy = pos.y/100;
    var out = 100 * (tempy - tempx * tempx) * (tempy - tempx * tempx) + (1 - tempx) * (1 - tempx);
    return out;
}
```

We shall look at the Particle class first. First, we initialise the position and velocity of the particle, as well as the variables that will hold the best result and corresponding position at which we obtained the best result.
```
function Particle(origin){
    // Initialise
    this.position = createVector(origin.x, origin.y);
    this.velocity = createVector(random(),random());
    this.position_best = createVector(origin.x, origin.y);

    // Set best position for the particle to infinity, i.e. the largest possible
    this.result_best = Infinity;
    this.result = Infinity;
```

Next, the function that will evaluate the result of the objective function at the current point of the particle; and update the best result and corresponding position at which we obtained the best result for the particle.
```
this.eval = function(func){
    this.result = func(this.position);

    if (this.result < this.result_best){
        this.position_best = this.position
        this.result_best = this.result;
    }

}
```

Then we update the particle’s velocity based on the inertia, cognitive and social components. These three components steer it in different directions, but the hope is that, on average, it will drive the particle to the minimum point of the objective function over time.
```
this.updateVelocity = function(global_pos_best){

    // Update using the PSO formula
    // var w = 0.5;
    // var c1 = 1;
    // var c2 = 2;

    r1 = random();
    r2 = random();

    var c = c1*r1*(this.position_best.x-this.position.x);
    var s = c2*r2*(global_pos_best.x-this.position.x);
    this.velocity.x = w*this.velocity.x+c+s;

    r1 = random();
    r2 = random();

    var c = c1*r1*(this.position_best.y-this.position.y);
    var s = c2*r2*(global_pos_best.y-this.position.y);
    this.velocity.y = w*this.velocity.y+c+s;
}
```

Finally we update the position, keep the particle within the search space, and display the particle.
```
this.updatePosition = function(bounds){

    // Keep solution, and hence positions of particles within bounds

    this.position.add(this.velocity);

    if (this.position.x>ub.x){
        this.position.x=ub.x;
    }
    if (this.position.x<lb.x){
        this.position.x=lb.x;
    }

    if (this.position.y>ub.y){
        this.position.y=ub.y;
    }
    if (this.position.y<lb.y){
        this.position.y=lb.y;
    }
}

// Display!
this.display = function() {
    stroke(255,5);
    strokeWeight(8);
    fill(255,119,51,100);
    // console.log('Radius', size);
    ellipse(this.position.x, this.position.y, 8, 8);
}
```

Next we setup the p5.js canvas and initialise a swarm of particles.
```
function setup(){

    w = 0.5;
    c1 = 1;
    c2 = 2;

    //Attaching the canvas created to the dom with the id p5canvas
    var c = createCanvas(500,500);
    c.parent('p5canvas');

    background(222,60,75);
    // Starting point, not used
    origin = createVector(width/2,height/2);
    // Upper and lower bounds
    ub = createVector(500,500);
    lb = createVector(0,0);

    // Initialise variable that will store best result at each run - smaller the better
    global_best_result= Infinity;

    // Variable to store best minimized position thus far
    global_pos_best = createVector(0.0,0.0);

    // Swarm of particles
    swarm = [];

    num_particles = 300;
    
    // Initialise
    for (var i=0; i<num_particles; i++){
        var newPos = createVector(random(margin,random(width-margin)),random(margin,random(height-margin)));
        var particle = new Particle(newPos);
        swarm.push(particle);
    }
}
```

And at each draw frame, we find the best global result (i.e. the best result for the swarm of particles), and feed that into the updateVelocity function within the Particle class.
```
var frameCount = 0;

function draw(){


    frameRate(10);
    background(222,60,75,255);

    // Evaluate the function by passing in the current position to function
    // Then compare and update global_pos_best if the result is < global_best_result so far
    for (var j=0; j<num_particles; j++){
        swarm[j].eval(f1);

        if (swarm[j].result<global_best_result){
            global_pos_best = swarm[j].position;
            global_best_result = swarm[j].result;
        }
    }

    // Update velocity, position and display circles 

    for (var j=0; j<num_particles; j++){
        swarm[j].updateVelocity(global_pos_best);
        swarm[j].updatePosition(lb,ub);
        swarm[j].display();
    }

    frameCount++;

    // Show results and Reset after 120 frames
    // I've randomised the main constants for the PSO formula to vary the outcome

    if(frameCount%120==0){
        console.log('Results');
        console.log(global_pos_best);
        console.log(global_best_result);  
        background(222,60,75,200);
        w = random();
        c1 = random(1,4);
        c2 = random(1,4);
        swarm = [];
        for (var i=0; i<num_particles; i++){
            var newPos = createVector(random(margin,random(width-margin)),random(margin,random(height-margin)));
            var particle = new Particle(newPos);
            swarm.push(particle);
        }
    }
}
```

I also wrote a simple if condition to reset the particles to random positions and start searching again after 120 iterations, just so that we can see how the particles move slightly differently for each round of iteration.

As this is a stochastic process, there is no guarantee that the global minimum of the objective function will be found, and the results may vary a little each time.

And that’s its for this very simple, but yet intriguing optimisation methodology. It supposedly gives pretty good results, comparable to the popular Stochastic Gradient Descent optimisation methodology used in neural networks.

The full code can be found [here][5]; and the simulation [here][6].

[1]:	https://github.com/playgrdstar/flocking_threejs
[2]:	https://playgrdstar.github.io/flocking_threejs/
[3]:	https://playgrdstar.github.io/particle_swarm_optimisation_p5/
[4]:	https://github.com/playgrdstar/particle_swarm_optimisation_p5
[5]:	https://github.com/playgrdstar/particle_swarm_optimisation_p5
[6]:	https://playgrdstar.github.io/particle_swarm_optimisation_p5/