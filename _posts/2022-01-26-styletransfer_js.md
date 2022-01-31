---
layout: default
title:  Style Transfer X Generative Art
description: Style Transfer X Generative Art
date:   2022-01-26 00:00:00 +0000
permalink: /style_transfer_gen_art/
category: Generative Art
---
## Style Transfer X Generative Art

Style transfer, where you transfer an artistic style to a selected image is great fun. 

But I thought that I would go one step further. Instead of just applying style transfer to some boring photo or image, I thought I would try to dynamically apply it to art generated that is continuously generated in the browser. For the generative art piece, I used the piece I generated previously and covered [here][1].

What I did was to capture the image from this piece every few seconds, and apply a style transfer model (trained on the Great Wave by Hokusai) on the generated art image.

The code [here][2] is pretty self explanatory for those who are interested. 

I only had a little trouble with converting the captured image into something that the style transfer model could work with. This little code snippet does the trick. 
```
function learn(){

    $('#outimage').empty();
    $('#styled').empty();
    $('#styled2').empty();
    saveFrames('_', '_', 1, 25, function(data) {
    // console.log(data[1].imageData);
    capturedImage = createImg(data[1].imageData, runModel);
    capturedImage.parent('outimage');
    });
}
```

First, I clear out all the images from previous runs of the style transfer model with jQuery’s empty() function. Then I use p5.js’ saveFrames function to capture the image on screen. This is in imageData (base64 binary) format so it won’t work if you just feed it into the style transfer mode. What I did was to first create a DOM image element from this binary. This can then be fed into the style transfer model, which is called using the runModel function.
```
function runModel(){
    console.log('Running model');
    style1.transfer(capturedImage, function(err, result) {
    createImg(result.src).parent('styled');
    createImg(result.src).parent('styled2');
    });
}
```

Check out the actual demo, running on p5.js and ml5.js, [here][3]. 

[1]:	https://medium.com/creative-coding-space/meet-blobby-in-p5-js-5d9d99232400
[2]:	https://github.com/playgrdstar/styletransfer_x_generativeart
[3]:	https://playgrdstar.github.io/styletransfer_x_generativeart/
