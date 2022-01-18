---
layout: default
title:  3 Days of Hand Coding Visualisations - Day 3
description: Series of tutorials on hand coding data visualisations with Javascript and D3.js
date:   2021-01-16 00:00:03 +0000
permalink: /handcode_viz_day_3/
category: Visualisation
---

## 3 Days of Hand Coding Visualisations - Day Three


_All code used in this set of tutorials is available at this github [link][1]. You can either download the entire repo, or just use the direct links to the files that I will provide at each step._

Yesterday, we finally managed to get our hands wet with d3.js and p5.js.

Today, we shall create a proper chart in d3.js. The file we shall focus on is [here][2], and the full file where I have gone further to create a number of other charts can be found [here][3]. 

The basic concept is not very different for the other charts, and I thought I would just include them here as a bonus (but I will try to cover some of the other charts in subsequent posts).

We shall build on what we did yesterday. Let’s skip the styling for now (if you remember, these come within the `<style></style>` tags).

We place a selector right at the top of our page. This allows us to choose the data file that we will load later. No need to understand too much of this, and I will cover this subsequently. Just know that this code produces the selector that you can use to choose the ticker value you want to select later to display.
```
var tickers = [{ticker:'EURUSD'}, 
			{ticker:'GBPUSD'}, 
			{ticker:'USDJPY'}]
var tickerselector = d3.select('#tickerselector');
tickerselector.append('select')
			.selectAll('option')
			.data(tickers)
			.enter()
			.append('option')
			.attr('value', function(d){
				return d.ticker;
			})
			.text(function(d){
				return d.ticker;
			});
```
Now, let’s first set the boundaries of the chart - the margins between the edge of the pages, and the width and height.
```
var lcmargin = 30;
var lcwidth = 800 - 2*lcmargin;
var lcheight = 200 - 2*lcmargin;
```

Then, like the yesterday, we append a SVG canvas for the line plot.
```
var linesvg = d3.select('#d3line')
		.append('svg')
		.attr('width', lcwidth + 2*lcmargin)
		.attr('height', lcheight + 2*lcmargin)
	.append('g')
		.attr('transform', 
			'translate(' + lcmargin + ',' + lcmargin + ')');
```

The second part, where we `append('g)` is new. Basically we create a group within the SVG canvas for the line plot. This allows us to later place and move the entire plot as a group, instead of having to deal with each element within separately. 

Here we transform it left and down by the margin (`lcmargin`) we set earlier.

Next we set the title of the chart. Everything before `.text` is just used to set the position, font size and color.
```
linesvg.append('text')
	.attr('x', lcwidth/2)
	.attr('y', lcmargin-45)
	.style('text-anchor', 'middle')
	.style('fill', color1)
	.style('font-size', '10px')
	.text('Currency Rates');
```
Then we create a series of variables that acts as helpers.
```
var parseTime = d3.timeParse('%Y-%m-%d %H:%M:%S');
var lineX = d3.scaleTime().range([0,lcwidth]);
var lineY = d3.scaleLinear().range([lcheight,0]);
```
`parseTime` helps to format what are just plain text dates into actual date and time objects.
`lineX` and `lineY` will help to map our data to the constraints(range) of the screen (width and height). 

Now for the main star of the show.
```
var valueLine = d3.line()
			.curve(d3.curveCatmullRom)
			.x(function(d){return lineX(d.Date);})
			.y(function(d){return lineY(d.Price);});
```

This is the function that will later translate the data into a line plot. It will use the CatmullRom method to interpolate between points, and assign all dates to the x axis, and price values to the y axis.

Next, we set the functions to call the x and y axis. The ‘Bottom’ and ‘Left’ here refers to which side the labels will be on, and not the position of the axis. Each of these axis will have 5 ticks.
```
var xAxisCallLine = d3.axisBottom().ticks(5);
var yAxisCallLine = d3.axisLeft().ticks(5);
```

Now we declare the core function - _initialiseLine_ - which will use the functions above, feed in the data corresponding the ticker, and draw the line. The comments below describe what each of the code blocks do (all the lines with `//` next to them).
```
var initialiseLine = function(ticker){

// d3.csv helps to read the csv file. 
d3.csv('../data/' + ticker+'.csv', function(error, data){

// This catches any errors, e.g. if the data file cannot be read.
	if (error) throw error;

// This parses the data, using the parseTime function we created earlier. + converts the data into a number (otherwise it would be text)
	data.forEach(function(d){
		d.Date = parseTime(d.time);
		d.Price = +d.c;
	});

//This continues where we started earlier. We get the domain of the data (min and max), and map it to the range we sent earlier (i.e. the width and height of the screen).
	lineX.domain(d3.extent(data, function(d){
		return d.Date;
	}));
	lineY.domain([
		d3.min(data, function(d){return d.Price;}),
		d3.max(data, function(d){return d.Price;})
	]);

//This line just calls the x and y axis
	xAxisCallLine.scale(lineX);
	yAxisCallLine.scale(lineY);

//Here we use the linesvg function we set earlier and pass in the data from the csv file to draw a path. The other lines set the style of the line
	linesvg.append('path')
		.data([data])
		.attr('class', 'line')
		.attr('d', valueLine)
		.style('fill', 'none')
		.style('stroke', color1)
		.style('stroke-width', '1px');

//Here we place the x and y axis on the page
	linesvg.append('g')
		.attr('transform', 'translate(0,' + lcheight + ')')
		.attr('class', 'x axis')
		.call(xAxisCallLine);
	linesvg.append('g')
		.attr('class', 'y axis')
		.call(yAxisCallLine);
		});
}
```
We have another function _updateLine_ that does the exact same thing, but we set it here to refresh the page when a new ticker is selected. I won’t go into the details as the code is almost identical to the one above. There is a easier way to do this, but I wanted to keep the two separate so that it’s clearer for you how they work later.

You can skip the next few lines. It’s pretty much the same, but the code is used to draw the dots at each data-point.

And now we call the functions to draw the line and dot plots.
```
initialiseLine('EURUSD');

tickerselector.on('change', function(){
	ticker = d3.select(this).select('select').property('value');
	updateLine(ticker);
})
```
And that’s it. I know that this is a lot to consume. I could have gone into a lot more detail on each and every line, but I always think that it’s good to jump right in, mess around with the code and get a feel for it.

And if you hit any roadblocks, just Google, or add a comment below and I will answer your question. 

Trust me, anything learnt this way will really stick!

[1]:	https://github.com/playgrdstar/handcoding_viz
[2]:	https://github.com/playgrdstar/handcoding_viz/blob/master/src/lab.html
[3]:	https://github.com/playgrdstar/handcoding_viz/blob/master/src/three_dashboard.html