---
layout: default
title:  3 Days of Hand Coding Visualisations - Day 2
description: Series of tutorials on hand coding data visualisations with Javascript and D3.js
date:   2021-01-16 00:00:02 +0000
permalink: /handcode_viz_day_2/
category: Visualisation
---

## 3 Days of Hand Coding Visualisations - Day Two


_All code used in this set of tutorials is available at this github [link][1]. You can either download the entire repo, or just use the direct links to the files that I will provide at each step._

Yesterday, we literally hand coded a visualisation. We measured and placed lines, circles and text to form a basic chart. The good thing is that we now know any chart is simply a collection of shapes. The bad thing is that this is not how we want to spend our lives!

And that is really why libraries exist. So that we can build on what someone else has already spent a lot of time on.

**Languages and Libraries**
The endgame is to code a chart in d3.js. But before we get into how to code a chart in d3.js, it’s probably good to know what else there is first.

- Languages - d3.js is Javascript library. There are other options. We can also code visualisations in Python using Matplotlib. And we can even have interactive charts when we code in environments like a Jupyter Notebook and make use of widgets.
- Libaries - d3.js is a library use SVG elements. We can also use libraries like Chart.js or p5.js which uses the HTML5 canvas element. Using SVG gives us greater flexibility in manipulating each element, whereas using canvas is a lot faster, and recommended if you have a lot of data points to render. It’s also possible to use canvas elements within d3.js. One more thing to note is that d3.js is a fairly low level library. So you actually still need to do a lot of the basic stuff and piece plots together with their axis, labels etc. There are also higher level libraries, like vega.js that make it a lot easier.

**What else is there?**
Certainly not the focus of this series, but for the curious, I have put together a series of examples on coding visualisations in:
- Python - In this [notebook][2], I used Python to visualise the returns of a range of FX currencies and futures contracts. We go beyond the usual line plots and bar charts and visualise data radially.
- Plotly - Plotly is a very popular library and online service that you can use Python or R to produce beautiful interactive Javascript charts for the web without knowing Javascript. This [notebook][3] shows how to use the basic features in Plotly.
- Vega.js - This is just one example of a higher level Javascript chart library. If I’m not wrong, it’s built on d3.js. Instead of having to write out Javascript, you use a standard set of vega.js codes in a [html][4] file, and then use a [json][5] file to set out the data and charts that you would like to visualise.


**Let’s start!**
Let’s do quick recap of HTML and CSS by going through the code of this basic page available [here][6].

There really is no need to retype out these things everytime. Just use this as the base. 

- `<!DOCTYPE html>` states that this is a HTML 5 file
- `<meta charset="utf-8">` tells the browser the character set this file uses
- `<head></head>` is where we place things we need but will not show up on the webpage
- `<style></style>` is where we state how we want the page styled. In this instance, we stated that elements with the class `.redtext` should have a background color of red
- `<body></body>` is where what we want seen on the webpage goes
- `<h1></h1>` is for the header text
- `<div><p>...</p></div>` is where we place the text we want. div tag defines a section; whereas p tag gets it to paragraph text
- `<h2 class='redtext></h2>` means that we set a class of ‘red text’ to whatever is within the tags; and it’s style is dictated by what we set within `<style></style>` earlier.

```
<!DOCTYPE html>
<meta charset="utf-8">
<head>

<style>
.redtext{
background-color: red;
}
</style>
</head>

<body>

<h1 class='redtext'>This is a webpage title</h1>
<h1> This is another title </h1>

<div><p>This is a para</p></div>
<h2 class='redtext' id='titleid'>This is big text</h2>
<h6 class='redtext' >This is big text</h6>

<script>
</script>
</body>
```

Next, from this basic template, we move on to see how to use the d3.js, as well as the p5.js libraries. The code that we will be referring to can be found [here][7].

You will also need the [libs][8] found here if you need to run the code.

First, let’s take a look at how we import the libraries that we need.
```
<head>
<!-- Bootstrap used for the positioning divs -->
<!-- CSS -->
<link rel="stylesheet" href="./lib/bootstrap3/css/bootstrap.min.css">
<link rel="stylesheet" href="./lib/css/font-awesome.min.css">

<!-- JavaScript - Local -->
<script src="./lib/jquery/jquery-3.2.1.min.js"></script>
<script src="./lib/bootstrap3/js/bootstrap.min.js"></script>
<script src="./lib/d3/d3.min.js"></script>

<!-- JavaScript - CDN -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/0.5.14/p5.js"></script>
```

We can import libraries from a location on your computer (local); or from the interwebs (usually using a CDN - content delivery network). Here we show how we can do both.

`<script src="./lib/d3/d3.min.js"></script>`

This just means that we go up one level (from where this code is, under the src folder), and look for the lib folder and access the d3.min.js file inside the d3 folder. Once we do this, we have access to all the functions in the d3.js library. Here we are using the minified library (which is faster to load).

We do this for all the other libraries and stylesheets. Here we use Bootstrap (which we can cover in another post) as well, so we import those as well, along with the jQuery library (which is needed for Bootstrap).

For the styling of the page, we just add a few lines to allow for the div sections to be entered and sized automatically.
```
<style>
/*To center (horizontally and vertically) the contents of the div*/
div {
	display: flex;
	justify-content: center;
	align-items: center;
}
</style>
```

**d3.js**
Next we set out two sections and give them each a unique ID. One will be for the d3.js visualisation, and the other will be for the p5.js one.
```
<div class="col-sm-12" id='tom'>
// Our d3.js visualisation will go here.
</div>

<div class="col-sm-4" id = 'p5canvas'>
// Our p5.js visualisation will go here.
</div>
```

And now for d3.js. What we will do first declare the width and height variables and set them to values of 400.

Then we select the section div with the id of ‘tom’, and append an SVG element to it, and set its width and height with this line.

```
var width = 400, height = 400;
var svg = d3.select('#tom')
		.append('svg')
		.attr('width', width)
		.attr('height', height);
```

Next we append a circle to this SVG canvas. We also set it’s properties.
```
var vectorcircle = svg.append('circle')
				.attr('cx', width/2)
				.attr('cy', height/2)
				.attr('r', 100)
				.style('fill', 'orange')
				.style('stroke', 'blue')
				.style('stroke-width', '3px')
```

When you refresh the page, you should now see a circle:
- right at the centre of the page (as cx is half the width of the page, and cy is half the height)
- radius of 100
- fill color orange
- stroke that is blue, with a thickness of 3px

And there, you have produced your first element in d3.js.

**p5.js**
It’s a lot simpler in p5.js. p5.js was designed for artists to code their creations, but it can easily be used for visualising data as well.

Every p5.js code (or sketch as it is usually called), has a setup and a draw function:
- the setup function is where you set out the basic outline of your sketch - the size of the sketch, the basic background color.
- the draw function is obviously where you draw to the screen. Unless we declare a noLoop on the setup function, draw will keep running and drawing to the screen as long as you are on the page.

In the setup function, we first create a canvas with the same height and width as the SVG canvas, and give it a background color of white.
```
function setup() {
	var canvas = createCanvas(width, height);
	canvas.parent('p5canvas');
	background('white');
}
```

And then we draw a circle, by stating that it’s fill should orange, and that it should have no stroke outline.

```
function draw(){
	fill('orange');
	noStroke();
	ellipse(width/2, height/2, 100, 100);
}
```

And that’s it! You are now able to create stuff in a browser using the d3.js and p5.js libraries.

[1]:	https://github.com/playgrdstar/handcoding_viz
[2]:	https://github.com/playgrdstar/handcoding_viz/blob/master/src/three_fancyhistograms.ipynb
[3]:	https://github.com/playgrdstar/handcoding_viz/blob/master/src/four_plotly.ipynb
[4]:	https://github.com/playgrdstar/handcoding_viz/blob/master/src/five_vega.html
[5]:	https://github.com/playgrdstar/handcoding_viz/blob/master/src/five_vega.json
[6]:	https://github.com/playgrdstar/handcoding_viz/blob/master/src/six_skeleton.html
[7]:	https://github.com/playgrdstar/handcoding_viz/blob/master/src/six_d3_p5.html
[8]:	https://github.com/playgrdstar/handcoding_viz/tree/master/lib