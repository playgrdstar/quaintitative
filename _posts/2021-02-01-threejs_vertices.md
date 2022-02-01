---
layout: default
title:  Three.js Series - Manipulating vertices in three.js
description: Three.js Series - Manipulating vertices in three.js
date:   2021-02-01 00:00:03 +0000
permalink: /vertices_threejs/
category: Generative Art

---
## Manipulating vertices in three.js

We had a simple primer on three.js [here][1].

Now, let’s do something a little more fancy, but not overly complicated.
It also gives us a good idea of how to objects are created in three.js and how to manipulate them at the vertice-level.

The code is almost identical to what we have previously. We ..
- set up the scene
- add the camera
- add the renderer
- attach it to the DOM
- add lights
- add spheres
- render and animate

We just do one more step within the animation loop.
```
sphere_one.geometry.vertices.forEach(function(i){
                    var noisy = noise.simplex3(i.x,i.y,i.z)*0.05;
                    i.x+=noisy*direction;
                    i.y+=noisy*direction;
                    i.z+=noisy*direction;
                });
```

What we are doing here is to run through each of the vertices of the sphere, and add a small noise to its x, y and z position. This causes the sphere to be a jaggy blob.

We also make one more small change here, which is to check the time elapsed since the start, and change the direction of the noise when it hits 100.
```
if (Math.floor(clock.getElapsedTime())%100==0){
                    direction*=-1;
                };
```

The full code is [here][2].


[1]:	https://github.com/playgrdstar/threejs_basics/blob/master/index.html
[2]:	https://github.com/playgrdstar/three_vertices/blob/master/index.html