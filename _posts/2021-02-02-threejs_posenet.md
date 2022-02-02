---
layout: default
title: Introduction to PoseNet with three.js
description: Introduction to PoseNet with three.js
date:   2021-02-02 00:00:00 +0000
permalink: /posenet_threejs/
category: Generative Art

---
## Introduction to PoseNet with three.js

Google has released the _tensorflow.js_ version of PoseNet for a while now. 

PoseNet is essentially a pre-trained convolutional neural network (CNN) that is able to detect a human pose in any image. This [paper][1] provides the technical details on PoseNet.

This _tensorflow.js_ version of PoseNet is actually really simple and straight-forward to use if you have any experience with Javascript. 

All you need to do is to load PoseNet from a CDN, use the browser to access the webcam, feed the webcam’s images into the trained PoseNet model, and it will spit out all the data on the locations of your body parts - eyes, ears, etc. 

You can then use the x and y coordinates of each part of your body for anything.

I’m fairly amazed at the ease of use, and the potential applications of PoseNet. 

Before this, one had to buy a XBOX Kinect, jump through some loops to get the drivers up, and then figure out the data coming out from the Kinect. Now that I have played around with PoseNet, I can see why Microsoft discontinued the Kinect. And I can certainly think about selling off my Kinect.

Actually, all one needs to do to use PoseNet is to go to the _tensorflow.js_ Github site and copy and paste the code of the entire demo there. 

But I thought it would be good to provide an even more gentle introduction to PoseNet, so I’ve simplified the main code from _tensorflow.js_ Github site’s official demo considerably. And instead of just rendering the position of each body part in HTML canvas (which is what the _tensorflow.js_ Github site does), I decided to use three.js spheres to track the body parts. I figured that it would be more fun to feel like you are moving a 3 dimensional object through virtual space.

**Before, we start, take a look at the demo [here][2]. **

You will have to grant access to your webcam. Don’t worry about that. As all the computing is being carried out on your browser, I’m not saving your images to any server anywhere. In this case, what happens on your browser, stays on your browser (which is kind of another side advantage of doing such stuff directly in your browser). I’ve also blurred and de-saturated the feed from the webcam.

So first off, in our HTML file, we insert the CDN links to the necessary libraries.
```
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/97/three.js"></script>
<script src="https://unpkg.com/@tensorflow/tfjs"></script>
<script src="https://unpkg.com/@tensorflow-models/posenet"></script>
```

And then we set up the divs for the webcam video, to draw the video to screen, and for the three.js canvas.
```
<div class='flex'>
    <div id='main' style='display:none'>
        <video id="video" playsinline style=" -moz-transform: scaleX(-1);
        -o-transform: scaleX(-1);
        -webkit-transform: scaleX(-1);
        transform: scaleX(-1);
        display: none;
        ">
        </video>
        <canvas id="output" />
    </div>  
</div>

<div id='threeContainer' class='flex'>
</div>
```

Now we move on to the main part - sketch.js.

We first set up the three.js scene.
```
// three.js setup

const width = 250;
const height = 250;

// Setup scene
const scene = new THREE.Scene();

//  We use an orthographic camera here instead of persepctive one for easy mapping
//  Bounded from 0 to width and 0 to height
// Near clipping plane of 0.1; far clipping plane of 1000
const camera = new THREE.OrthographicCamera(0,width,0,height, 0.1, 1000);
camera.position.z = 500;

// Setting up the renderer
const renderer = new THREE.WebGLRenderer( { antialias: true } );
renderer.setPixelRatio( window.devicePixelRatio );
renderer.setSize( width, height );
renderer.setClearColor( 0xDE3C4B, 1 );

// Attach the threejs animation to the div with id of threeContainer
const container = document.getElementById( 'threeContainer' );
container.appendChild( renderer.domElement );

// Scene lighting
const hemiLight     = new THREE.HemisphereLight('#EFF6EE', '#EFF6EE', 0 );
hemiLight.position.set( 0, 0, 0 );
scene.add( hemiLight );

const group = new THREE.Group();
```

I create a Tracker class to help me in creating and moving spheres on the screen.
```
// Creating Tracker class
function Tracker(){
    this.position = new THREE.Vector3();

    const geometry = new THREE.SphereGeometry(10,7,7);
    const material = new THREE.MeshToonMaterial({ color: 0xEFF6EE, 
                                                opacity:0.5, 
                                                transparent:true, 
                                                wireframe:true, 
                                                emissive: 0xEFF6EE,
                                                emissiveIntensity:1})

    const sphere = new THREE.Mesh(geometry, material);
    group.add(sphere);

    this.initialise = function() {
    this.position.x = -10;
    this.position.y = -10;
    this.position.z = 0;
    }

    this.update = function(x,y,z){
    this.position.x = x;
    this.position.y = y;
    this.position.z = z;
    }

    this.display = function() {
    sphere.position.x = this.position.x;
    sphere.position.y = this.position.y;
    sphere.position.z = this.position.z;

    // console.log(sphere.position);
    }
}
```

And add them to the scene.
```
scene.add( group );
```

Now we move on to the parts that load the PoseNet model, feed in the webcam images, and get at the positions of the body parts. I will just go into the essential parts here.

First, the functions to setup the camera and load the video.
```
// Load camera
async function setupCamera() {
    if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
    throw new Error(
        'Browser API navigator.mediaDevices.getUserMedia not available');
    }

    const video = document.getElementById('video');
    video.width = width;
    video.height = height;

    const mobile = isMobile();
    const stream = await navigator.mediaDevices.getUserMedia({
    'audio': false,
    'video': {
        facingMode: 'user',
        width: mobile ? undefined : width,
        height: mobile ? undefined : height,
    },
    });
    video.srcObject = stream;

    return new Promise((resolve) => {
    video.onloadedmetadata = () => {
        resolve(video);
    };
    });
}

async function loadVideo() {
    const video = await setupCamera();
    video.play();

    return video;
}
```

I then initialise my 17 trackers (corresponding to the 17 body parts tracked by PoseNet).
```
// Initialise trackers to attach to body parts recognised by posenet model

let trackers = [];

for (let i=0; i<17; i++){
    let tracker = new Tracker();
    tracker.initialise();
    tracker.display();

    trackers.push(tracker);
}
```

Then we setup the main render and detect functions.
```
let net;

// Main animation loop
function render(video, net) {
    const canvas = document.getElementById('output');
    const ctx = canvas.getContext('2d');

    // Flip the webcam image to get it right
    const flipHorizontal = true;

    canvas.width = width;
    canvas.height = height;

    async function detect() {

    // Load posenet
    net = await posenet.load(0.5);

    // Scale the image. The smaller the faster
    const imageScaleFactor = 0.75;

    // Stride, the larger, the smaller the output, the faster
    const outputStride = 32;

    // Store all the poses
    let poses = [];
    let minPoseConfidence;
    let minPartConfidence;

    const pose = await net.estimateSinglePose(video, 
                                                imageScaleFactor, 
                                                flipHorizontal, 
                                                outputStride);
    poses.push(pose);

    // Show a pose (i.e. a person) only if probability more than 0.1
    minPoseConfidence = 0.1;
    // Show a body part only if probability more than 0.3
    minPartConfidence = 0.3;

    ctx.clearRect(0, 0, width, height);

    const showVideo = true;

    if (showVideo) {
        ctx.save();
        ctx.scale(-1, 1);
        ctx.translate(-width, 0);
        // ctx.filter = 'blur(5px)';
        ctx.filter = 'opacity(50%) blur(3px) grayscale(100%)';
        ctx.drawImage(video, 0, 0, width, height);
        ctx.restore();
    }

    poses.forEach(({score, keypoints}) => {
        if (score >= minPoseConfidence) {
        keypoints.forEach((d,i)=>{
            if(d.score>minPartConfidence){
            // console.log(d.part);
            // Positions need some scaling
            trackers[i].update(d.position.x*0.5, d.position.y*0.5-height/4,0);
            trackers[i].display();
            }
            // Move out of screen if body part not detected
            else if(d.score<minPartConfidence){
            trackers[i].update(-10,-10,0);
            trackers[i].display();
            }
        })
        }
    });

    renderer.render( scene, camera );
    requestAnimationFrame(detect);
    }

    detect();

}
```

I’ve provided comments on the key parts of the code above, so you should not have much difficulty understanding these lines. This part of the code in the official demo is fairly more complicated and offers more options, but I’ve rewritten this, and taken out a lot of the extra lines to allow one to more easily understand the how to load and use PoseNet.

Now we just run all the functions in the main function.
```
async function main() {
    // Load posenet
    const net = await posenet.load(0.75);

    document.getElementById('main').style.display = 'block';
    let video;

    try {
    video = await loadVideo();
    } catch (e) {
    let info = document.getElementById('info');
    info.textContent = 'this browser does not support video capture,' +
        'or this device does not have a camera';
    info.style.display = 'block';
    throw e;
    }

    render(video, net);
}

navigator.getUserMedia = navigator.getUserMedia ||
    navigator.webkitGetUserMedia || navigator.mozGetUserMedia;


main();
```

And that’s it. The full code is available [here][3].

[1]:	https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/42237.pdf
[2]:	https://playgrdstar.github.io/posenet_threejs/
[3]:	https://github.com/playgrdstar/posenet_threejs