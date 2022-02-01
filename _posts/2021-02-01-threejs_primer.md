---
layout: default
title:  Three.js Series - Simple primer on three.js 
description: Three.js Series - Simple primer on three.js 
date:   2021-02-01 00:00:01 +0000
permalink: /primer_threejs/
category: Data Engineering

---
## Simple primer on three.js 

Compared to Processing or p5.js, three.js may seem a little intimidating at the start. The main reason for that is really just the code needed to set-up the 3D scene. Other than that, it really isn’t much more complicated. 

The three.js documentation is great, but the example provided within might not go far enough.

So what I’ll do here is to use a three.js piece that I did that goes a little further than the example on the three.js site, and use that to explain some key concepts in three.js. The complete code is available [here][1].

First, we import the core three.js library as well as the OrbitControls.js library. The three.js library has the core functions, while the  OrbitControls.js library will allow us to insert controls to zoom, pan and rotate the view.
```
<script type="text/javascript" src="./three.js"></script>
<script type="text/javascript" src="./OrbitControls.js"></script>
```

Setup the scene by declaring -
- a scene `var scene = new THREE.Scene();`
- a camera `var camera = new THREE.PerspectiveCamera( 100, window.innerWidth / window.innerHeight, 10, 1000 );`
- a renderer `var renderer = new THREE.WebGLRenderer( { antialias: true } );`
  
```
var scene = new THREE.Scene();
var camera = new THREE.PerspectiveCamera( 100, window.innerWidth / window.innerHeight, 10, 1000 );
camera.position.z = 30;
var renderer = new THREE.WebGLRenderer( { antialias: true } );
renderer.setPixelRatio( window.devicePixelRatio );
renderer.setSize( window.innerWidth, window.innerHeight );
renderer.setClearColor( 0xCC7FE0, 1 );

document.body.appendChild( renderer.domElement );
```

The other stuff within the code above are used to set the options for these variables, e.g. setting the position of the camera on the z axis `camera.position.z = 30;`. The last line is the most important. It appends the renderer to the body of the HTML document so that we can see whatever we are drawing to the screen.

Next we declare an instance of the OrbitControl.
```
var orbit = new THREE.OrbitControls( camera, renderer.domElement );
orbit.enableZoom = false;
```

And declare and add some lights to the scene.
```
var ambientLight  = new THREE.AmbientLight( '#FFE7FF' ),
    hemiLight     = new THREE.HemisphereLight('#FFE7FF', '#FFE7FF', 0.5 ),
    light         = new THREE.PointLight( '#FFE7FF', 1, 100 );

hemiLight.position.set( 0, 0, 10 );
light.position.set( 0, 0, 0 );

scene.add( hemiLight );
scene.add( light );
```

Notice that we add an item to the scene each time we create it. The ‘set’ function is really just for setting the x, y, and z coordinates of the each item that we add to the scene (light, camera).

Now we start creating the objects within the scene. Let’s add one sphere.
```
var group = new THREE.Group();
var geometry = new THREE.SphereGeometry( 3, 32, 32 );
var material = new THREE.MeshBasicMaterial( { color: 0xffffff, opacity:0.7, transparent:true } );
var sphere_one = new THREE.Mesh( geometry, material );
sphere_one.position.x = 0;
sphere_one.position.y = 0;
sphere_one.position.z = 0;

group.add( sphere_one );
```

The lines above -
- declare a new group - this allows us to create subgroup of items within the scene
- declare a new geometry for sphere with a radius of 3, and 32 faces
- declare a new basic material with the color white (#ffffff), that is transparent with an opacity of 0.7
- and with the geometry and material create a sphere.

We then set the position of the sphere and add it to the group.

The script does the same for another 2 spheres, with different materials.

We then add the whole group into the scene.
```
scene.add( group ); 
```

Next we render the scene and animate it.
```
var render = function () {
                requestAnimationFrame( render );
                orbit.update();
                group.rotation.x += 0.03;
                group.rotation.y += 0.02;
                group.rotation.z += 0.01;
                
                renderer.render( scene, camera );
            };
```

At each frame, the group of items is rotated.

The full code is [here][2].


[1]:	https://github.com/playgrdstar/threejs_basics/blob/master/index.html
[2]:	https://github.com/playgrdstar/threejs_basics/blob/master/index.html