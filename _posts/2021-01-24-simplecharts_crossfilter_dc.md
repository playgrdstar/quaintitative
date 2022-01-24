---
layout: default
title:  Simple charts | crossfilter.js and dc.js
description: Experiments in visualisation - Simple interactive charts with crossfilter.js and dc.js
date:   2021-01-24 00:00:00 +0000
permalink: /crossfilter_dc_js/
category: Visualisation
---
### Experiments in visualisation - Simple interactive charts with crossfilter.js and dc.js

_ What this covers: Javascript, D3.js, crossfilter.js, dc.js_

I have always liked doing things the hard way. And so I have eschewed the higher level visualisation libraries that allow one to create charts in a few lines.

D3.js has hence been the tool of my choice for online visualisation for a while now (and will be the subjects of other posts). But decided to experiment with the [crossfilter.js][1] and [dc.js][2] libraries recently.

First-off, what do the crossfilter.js and dc.js libraries do?
- **crossfilter.js** makes aggregation and manipulation of data in Javascript painless. For those familiar with Excel PivotTables, this is basically its counterpart in code. For those who are familiar with Tableau or Qlikview/Qliksense, this works in a very similar way. It allows one to create dimensions and perform computations on subsets of data within these dimensions. On its own, crossfilter.js only helps one to manipulate data.
- But when combined with **dc.js**, a high level charting library, you gain the powers (for free) to code out online visualisations that come at a hefty cost in Tableau or Qliksense.

Now for a demonstration. Full code for the demonstration is available [here][3].

I shall not explain the HTML and CSS boilerplate here (but there are comments within the code for a newbie).

First, the data. The mock data that I have used here are meant to represent asset returns for a portfolio from 2014-2018. Flat data is required here, and it will be clear why later.
```
var portfolioData = [
    {Year: 2014, Asset: 'Equity', Return: 0.045},
    {Year: 2014, Asset: 'Bond', Return: 0.025},
    {Year: 2014, Asset: 'Gold', Return: 0.015},
    {Year: 2014, Asset: 'Cash', Return: 0.005},

    ...
];
```
Next we feed the data to the crossfilter function.
```
var datax = crossfilter(portfolioData);
```

If you do a `console.log(datax)` now, you will see that all matters of functions are available to you.

For the purposes of this tutorial, we want to look at the **Year** and **Asset** dimensions, and sum the **Return** by these dimensions.
```
var yearDimension = datax.dimension(function(d) {return d.Year;});
var assetDimension = datax.dimension(function(d) {return d.Asset;});
var returnsPerYear = yearDimension.group().reduceSum(function(d) {return +d.Return;});
var returnsPerAsset = assetDimension.group().reduceSum(function(d) {return +d.Return;});
```

That’s basically all we need to do for data processing. Everything is now set-up for the charts.

We will do two charts, one for the year on year returns (as a row chart, or rather a vertical bar chart); and one to show how much each asset contributed to returns for the year.

First the row chart. The code for this part is really simple -
- Get `returnsRowChart` setup with `dc.rowChart` function
- Feed in the dimension and the sum of returns by year
- Tweak the way the x axis is shown
- Choose the colors
```
var returnsRowChart = dc.rowChart("#rowchart");

returnsRowChart
    .dimension(yearDimension)
    .group(returnsPerYear)
    .controlsUseVisibility(true);

returnsRowChart.xAxis().ticks(5).tickFormat(function(d){return d*100+'%';});

returnsRowChart.ordinalColors(['#ffc0cb','#ff99a2', '#ff7379', '#ff4c51', '#ff2628']);
```

That’s it!

Next, let’s do the pie chart. The steps are very much similar. 
```
var returnsPieChart = dc.pieChart("#piechart");

returnsPieChart
        .dimension(assetDimension)
        .group(returnsPerAsset)
        .innerRadius(50)
        .controlsUseVisibility(true)
        .on('pretransition', function(chart) {
            chart.selectAll('text.pie-slice').text(function(d) {
                return d.data.key + '-' + dc.utils.printSingleValue((d.endAngle - d.startAngle) / (2*Math.PI) * 100) + '%';
            })
        });

returnsPieChart.ordinalColors(['#ffc0cb','#ff99a2','#ff4c51', '#ff2628']);
```

There is one line within that deserves a little more explanation. What this does is to show the percentage by using the end and start angles datapoint that were automatically created when you passed data into the pieChart function.
```
.on('pretransition', function(chart) {
            chart.selectAll('text.pie-slice').text(function(d) {
                return d.data.key + '-' + dc.utils.printSingleValue((d.endAngle - d.startAngle) / (2*Math.PI) * 100) + '%';
            })
        });
```
And finally…
```
dc.renderAll();
```

This final line renders the charts. You could also call `render()` on each chart individually, example -
```
returnsPieChart.render();
```

However, do note that the two charts will not be linked and will not dynamically update in relation to each other if you do this.

[1]:	http://square.github.io/crossfilter/
[2]:	http://dc-js.github.io/dc.js/
[3]:	https://github.com/playgrdstar/crossfilter_dc_demo