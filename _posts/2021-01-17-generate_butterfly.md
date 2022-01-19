---
layout: default
title:  Generating a Strange Attractor in three.js
description: Generating a Strange Attractor in three.js
date:   2021-01-17 00:00:01 +0000
permalink: /gen_strange_attractor/
category: Generative Art
---
## Generating a Strange Attractor in three.js

Art and math - An experiment with the lorenz function in three.js

Quantitative finance, data science, art, nature...

You would think that these are totally separate fields, but the deeper down the rabbit hole you go, the more you realise how much they have in common. Concepts relating to randomness and patterns, fractals, golden ratios span all these fields.

As I work in a finance field, dabble in data science, and love both illustration and generative art, these commonalities never fail to amaze me. 

More on this subject another day, but for now, a tutorial on how to do a visualisation using three.js and the Lorenz equation.

Edward Lorenz, developed the equation while modelling the weather, and is also credited with the ‘butterfly effect’ - which is basically an insight into how small changes in nature can have huge consequences. 

> Does the flap of a butterfly’s wings in Brazil set off a tornado in Texas?  - Title of Edward Lorenz’s 1972 paper

These are all related to this concept of 'Strange Attractors', of which the Lorenz equation or function is one. So let's try generating it.

First, we set up the scene, camera, lights, and the renderer to render them to the screen.
```
// Setting up the scene
var scene = new THREE.Scene();

// Setting up a camera
var camera = new THREE.PerspectiveCamera( 100, window.innerWidth / window.innerHeight, 0.1, 50 );
camera.position.z = 30;

// Setting up the renderer. This will be called later to render scene with the camera setup above
var renderer = new THREE.WebGLRenderer( { antialias: true } );
renderer.setPixelRatio( window.devicePixelRatio );
renderer.setSize( window.innerWidth, window.innerHeight );
renderer.setClearColor( 0xF70051, 1 );
document.body.appendChild( renderer.domElement );

// Setting up a light
var light = new THREE.PointLight( '#9BC995', 1, 1000 );
light.position.set( 0, 0, 0 );
scene.add( light );
```
We then generate the points plotting out the familiar butterfly shape of the Lorenz Equation.
```
function lorenz(){

    var arrayCurve=[];

    var x = 0.01, y = 0.01, z = 0.01;
    var a = 0.9;
    var b = 3.4;
    var f = 9.9;
    var g = 1;
    var t = 0.001;
    for (var i=0;i<100000;i++){

    var x1 = x;
    var y1 = y;
    var z1 = z;
    x = x - t*a*x +t*y*y -t*z*z + t*a*f; 
    y = y - t*y + t*x*y - t*b*x*z + t*g;
    z = z - t*z + t*b*x*y + t*x*z;
    arrayCurve.push(new THREE.Vector3(x, y, z).multiplyScalar(1));   
    }
    return arrayCurve;
}
```
These are fed into the geometry of a cloud of points.
```
// Array curve holds the positions of points generated from a lorenz equation; lorenz function below generates the points and returns an array of points
var arrayCurve = lorenz();

// Generating the geometry
var curve = new THREE.CatmullRomCurve3(arrayCurve);
var geometry = new THREE.Geometry();
geometry.vertices = curve.getPoints(100000);

// Generating a cloud of point
var pcMat = new THREE.PointsMaterial();
pcMat.color = new THREE.Color(0x5555ff);
pcMat.transparent = true;
pcMat.size = 0.05;
pcMat.blending = THREE.AdditiveBlending;
pc = new THREE.Points(geometry, pcMat);
pc.sizeAttenuation = true;
pc.sortPoints = true;
```
Now we illustrate the actual butterfly effect, by changing the coefficients in the Lorenz equation slightly and randomly for each frame.
```
var render = function () {

    renderer.render( scene, camera );
    requestAnimationFrame( render );

    //Varying the points on each frame
    step+=0.01;
    var count = 0;
    var geometry = pc.geometry;
    var a = 0.9+Math.random()*6;
    var b = 3.4+Math.random()*7;
    var f = 9.9+Math.random()*8;
    var g = 1+Math.random();
    var t = 0.001;


    geometry.vertices.forEach(function(v){
        v.x = v.x - t*a*v.x +t*v.y*v.y -t*v.z*v.z + t*a*f; 
        v.y = v.y - t*v.y + t*v.x*v.y - t*b*v.x*v.z + t*g;
        v.z = v.z - t*v.z + t*b*v.x*v.y + t*v.x*v.z;
    })
    geometry.verticesNeedUpdate = true;

    group.rotation.x += 0.01;
    group.rotation.y += 0.02;
    group.rotation.z += 0.01;
};
```
And that’s it! :)

The actual simulation of the Lorenz equation and the butterfly effect can be found [here][1]; and the code [here][2].

[1]:	https://playgrdstar.github.io/lorenz_threejs/
[2]:	https://github.com/playgrdstar/lorenz_threejs