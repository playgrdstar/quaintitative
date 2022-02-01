---
layout: default
title:  Three.js Series - Music and three.js
description: Three.js Series - Music and three.js
date:   2021-02-01 00:00:02 +0000
permalink: /music_threejs/
category: Generative Art

---
## Music and three.js

We had a simple primer on three.js [here][1], and one on manipulating vertices in three.js [here][2].

Now, let’s do something fancier. There is a reason for the term audiovisual, so let’s create something that is not only visually appealing, but moves with music.

The code here is very similar to the last one on manipulating vertices. However, we focus on just one sphere here. 

Let’s load the media (music) element, and add an Analyser instance to extract numerical data out of the music.
```
var fftSize = 128;
var listener = new THREE.AudioListener();
var audio = new THREE.Audio( listener );
var mediaElement = new Audio( 'music/Jester_IanPost.mp3' );
mediaElement.loop = true;
                
mediaElement.play();
audio.setMediaElementSource( mediaElement );
analyser = new THREE.AudioAnalyser( audio, fftSize );
```

With the Analyser instance, we can compute the average of the numbers with a simple reduce function.
```
var sum = analyser.data.reduce(function(a,b){return a+b;});
var avg = sum/analyser.data.length;
```

We can then use the average value to change the speed of change of the positions of each of the vertices, and the rotation of the sphere.
```
sphere_one.geometry.vertices.forEach(function(i){
var noisy = noise.simplex3(i.x,i.y,i.z)*0.0002;
                        i.x+=noisy*avg;
                        i.y+=noisy*avg;
                        i.z+=noisy*avg;
                    });

group.rotation.x += 0.0005*avg;
group.rotation.y += 0.0005*avg;
group.rotation.z += 0.0005*avg;
```

We need to update the Analyser during each frame.
```
analyser.getFrequencyData();
```

And that’s it. Enjoy!

The full code is [here][3].


[1]:	https://github.com/playgrdstar/threejs_basics/blob/master/index.html
[2]:	https://github.com/playgrdstar/three_vertices/blob/master/index.html
[3]:	https://github.com/playgrdstar/threejs_music/blob/master/index.html