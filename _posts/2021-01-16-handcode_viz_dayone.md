---
layout: default
title:  "3 Days of Hand Coding Visualisations - Day 1"
date:   2021-01-16 00:00:01 +0000
permalink: /handcode_viz_day_1/
---

## 3 Days of Hand Coding Visualisations - Day One


_All code used in this set of tutorials is available at this github [link][1]. You can either download the entire repo, or just use the direct links to the files that I will provide at each step._

As we are going to be showing off our creations in the web browser, we need to have an understanding of how a web page works. A basic understanding of HTML, CSS and Javascript is what I hope to provide in this Day One tutorial.

** HTML 101**
First, a super basic HTML page that we can easily use for coding a data visualisation looks like this.
```
<!DOCTYPE html>
<meta charset="utf-8">

<head>
<script src="../lib/d3/d3.min.js"></script>
<style>
</style>
</head>

<body>

</body>
```
Let’s break this down. It’s quite simple actually. Let’s call each of these words enclosed in \<\> tags.

The first two lines are not that critical to know. Just know that they are almost always the first two lines of a modern webpage.
- Right at the top, we have the tag `<!DOCTYPE html>` right at the top. This just tells the browser what version of HTML you are writing in. There’s no real need to understand this, but just know that this indicates a HTML 5 webpage. Anything else (which is usually much much longer) at the top of the page indicates that the webpage was written with an older version of HTML.
- The next tag `<meta charset="utf-8">` basically specifies what character set the webpage is written. Again no real need to know the technical details.

The next few lines are important.
- `<head>...</head>` This is an opening and closing tag that marks the head of the document. The head of the document is where you place meta information that is not visible on the webpage. It’s also where we set out libraries and styles that we will use for the webpage
- `<script>...</script>` Within the script tags, we can insert javascript codes, or links to javascript that we use within the webpage
- `<style>...</style>` Within the style tags, we can insert code that allows us to style the webpage, or links to a stylesheet. These are cascading style sheets, or what is known as CSS
- `<body>...</body>` is self explanatory. It’s where code for the body of our webpage, which is what we see when we visit any webpage, resides.

And that’s it. You now know HTML (or at least the rudimentary parts of HTML) needed to get things up and running.

The code for this first part is available [here][2].

**Now Draw a Circle in SVG**

SVG stands for Scalable Vector Graphics. Basically, it’s what we can use to draw things on a webpage. We can also use something called the Canvas to draw, but we will go into that later.

Now that you have your webpage skeleton in place, drawing a circle just takes one line.
```
<svg width="500" height="500">
	<circle cx="250" cy="250" r="40" stroke="black" stroke-width="3" fill="red"/>
</svg>
```
`<svg>...</svg>` tells the browser that we are creating an SVG object. We first declare a `<circle/>`. There’s no need for a closing tag here cause of the additional / at the end. With that, it’s a simple matter of declaring the properties of the circle - cx and and cy are its x and y coordinates, r is its radius, stroke, stroke-width and fill the color of its outline stroke, stroke thickness, and fill color respectively.

That’s it! You can now draw things in your browser. The code for this part is available [here][3]. 

**Using SVG to code a chart**

Now let’s do things the really hard way first, before we start learning about how to use libraries like D3.js.

Using the HTML skeleton you built earlier, insert the title and the size of the SVG object that we will be creating first between the body tags.
```
<h4 fill='black'>Barebones Chart</h4>
<svg width = "600" height = "600">
...
</svg>
```
`<h4>` is a header tag. There also h1, h2, all the way to h6. The only difference is the size. `<svg width='600' height='600'>` sets the width and height of the SVG canvas that we will be drawing things in 
 
Now, just copy and paste these lines between the `<svg></svg>` tags. 

For the axis of the line chart.
```
<g>
<line stroke='steelblue' x1="90" x2="90" y1="5" y2="371">
</line>
</g>

<g>
<line stroke='steelblue' x1="90" x2="705" y1="370" y2="370"></line>
</g>
```

For the labels.
```
<g transform='translate(0,10)'>
<text x="100" y="400">2013</text>
<text x="246" y="400">2014</text>
<text x="392" y="400">2015</text>
<text x="538" y="400">2016</text>
<text x="684" y="400">2017</text>
<text x="400" y="440">Dates</text>
</g>

<g transform='translate(-10,0)'>
<text x="80" y="15">60</text>
<text x="80" y="131">40</text>
<text x="80" y="248">20</text>
<text x="80" y="373">0</text>
<text x="40" y="200">Levels</text>
</g>
```

For the circles showing each datapoint.
```
<g>
<circle cx="90" cy="192" r="4" fill='steelblue'></circle>
<circle cx="240" cy="141" fill='steelblue' r="4"></circle>
<circle cx="388" cy="179" fill='steelblue' r="4"></circle>
<circle cx="531" cy="200" fill='steelblue' r="4"></circle>
<circle cx="677" cy="104" fill='steelblue' r="4"></circle>
</g>
```

And the line plotting out the data points.
```
<polyline
		fill="none"
		stroke="steelblue"
		stroke-width="1"
		points="
		90,192
		240,141
		388,179
		531,200
		677,104"/>
```

Insert the code step by step and you will see the chart slowly appear, like magic. To understand how the code works, just try changing the numbers and properties and it will become quite clear. This is not the most optimal way of hand coding visualisations so I won’t go into great detail. But it should be fairly obvious.

The code for this part is available [here][4]. 

This is obviously not the best way to hand code data visualisations. But it’s good to understand how things come together on a webpage. Otherwise, it’s a bit harder to grasp how libraries like D3.js work (which we will cover in the next 2 days).

And that’s it for the first day!

[1]:	https://github.com/playgrdstar/handcoding_viz
[2]:	https://github.com/playgrdstar/handcoding_viz/blob/master/src/one.html
[3]:	https://github.com/playgrdstar/handcoding_viz/blob/master/src/one_ans.html
[4]:	https://github.com/playgrdstar/handcoding_viz/blob/master/src/two.html