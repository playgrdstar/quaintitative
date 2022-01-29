---
layout: default
title:  Growing a network garden in D3
description: Growing a network garden in D3
date:   2021-01-29 00:00:01 +0000
permalink: /network_garden_d3/
category: Visualisation
---
## Growing a network garden in D3

Network analysis is a huge field of study in and of itself, but if we just need a quick easy way to visualise relationships, we can use D3 to quickly create a network diagram.

Other than a boring mess of lines and circles, I’ve used the ‘stroke-dasharray’ style property to create a network diagram that resembles a garden.

First we set up the constants.
```
// Color palette

var raspberry = '#8A1C59';
var magenta = '#FC60A8';
var bluejeans = '#5ECAE9';
var spacecadet = '#173753';
var powderblue = '#BEE9E8';

var flowercol1 = '#F815CD';
var flowercol2 = '#EB0A9F';
var flowercol3 = '#BA00C9';

var margin = 0;
var width = 800 - 2*margin, height = 800 - 2*margin;
var selectedarea;

var baseRadius = 40;
var alphaCoeff = 0.1

var textSizeOne = '22px';
var textSizeTwo = '12px';
```

Most of these should be familiar to you if you have looked at my previous posts, perhaps with the exception of alphaCoeff. This is just an input to a D3 function which determines the dampening for the network chart to settle down.

And then we run through the usual steps - creating the svg, textbook, etc.
```
var svg = d3.select('#d3canvas')
            .append('svg')
            .attr('width', width + 2*margin)
            .attr('height', height + 2*margin)
var g = svg.append('g')
            .attr('transform', 'translate(' + margin + ',' + margin + ')');


// Background color for the svg using a rect
g.append('rect')
    .attr('width', width + 2*margin)
    .attr('height', height + 2*margin)
    .attr('fill', powderblue);

// Title text
var textBox = d3.select('#d3text')
                .append('p');
textBox
    .append('h1')
    .attr('class', 'descriptor')
    .attr('x', 0)
    .attr('y', 0)
    .text('network garden')
    .style('color', magenta)
    .style('opacity', 1.0)
    .style("font-family", "sans-serif")
    .style('font-weight', 'bold')
    .style("font-size", "14px")
    .style('text-anchor', 'middle');
```

We add in an event handler for mouse scroll events to allow the viewer to zoom in; and also a reset function to reset the view.
```
//add zoom capabilities 
var zoom_handler = d3.zoom()
    .on("zoom", zoom_actions);
    
zoom_handler(svg);     
    
function zoom_actions(){
    g.attr("transform", d3.event.transform)
}


d3.select("#reset")
    .on("click", resetted);

function resetted() {
    svg.transition()
        .duration(750)
        .call(zoom_handler.transform, d3.zoomIdentity);
}
```

Now, we read and munge the data.
```
d3.csv('/data/network.csv', function(error, links){
    if (error) throw error;


    // To set colors and size based on tags
    links.forEach(function(link){
    if(link.TagTwo=='High'){
        link.col = flowercol1;
    } else if (link.TagTwo=='Medium'){
        link.col = flowercol2;
    } else if (link.TagTwo=='Low'){
        link.col = flowercol3;
    }
    })
    links.forEach(function(link){
    if(link.TagThree=='Completed'){
        link.sizeMul = 0.5;
    } else if (link.TagThree=='In Progress'){
        link.sizeMul = 2.5;
    }
    })
    var nodes = {};

    links.forEach(function(link){
    // link source or target will equal NodeOne or NodeTwo if it exists
    // OR it will create and assign a new node
    // We also add attributes to NodeTwo
    link.source = nodes[link.NodeOne] || 
                    (nodes[link.NodeOne] = {name: link.NodeOne});

    
    link.target = nodes[link.NodeTwo] || 
                    (nodes[link.NodeTwo] = {name: link.NodeTwo, tagone:link.TagOne, 
                                            tagtwo: link.TagTwo,
                                            tagthree:link.TagThree,
                                            col:link.col,
                                            sizeMul:link.sizeMul});
    });

var nodes = d3.values(nodes); // Returns an array of nodes - i.e. change nodes from an object to an array
```

And with that, we can move on to the meat of the code that draws the network/force diagram to the screen.
```
function drawChart(nodes, links, area) {

// D3 simulation
    var repelForce = d3.forceManyBody().strength(-200).distanceMax(500).distanceMin(100);

    var simulation = d3.forceSimulation()
                        .force('link', d3.forceLink().links(links))
                        .force('charge', d3.forceManyBody())
                        .force('repel', repelForce)
                        .force('center', d3.forceCenter(width/2, height/2))
                        .force('collision', d3.forceCollide().radius(baseRadius))


    simulation.alphaTarget(alphaCoeff); // dampening for the force chart to settle down
```	

Next we set up the nodes, links and text.
```
var nodetext = g.append('g')
            .attr('class', 'nodestext')
            .selectAll('circle')
            .data(nodes)
            .enter()
            .append('text')
                    .attr('class', 'nodestext-text')
            .text(function(d){return d.name;})
            .style("font-size", textSizeOne)
            .attr('font-weight', 'bold')
            .attr('fill', spacecadet)
            .attr('fill-opacity', 0.5)
            .style('pointer-events', 'none')
            .style('text-anchor', 'middle')
            .on('click', connectedNodes)
            // .on('mouseout', connectedNodes);

var node = g.append('g')
            .attr('class', 'nodes')
            .selectAll('circle')
            .data(nodes)
            .enter()
            .append('circle')     
            .attr('r', baseRadius)
            .attr('fill', bluejeans)
            .attr('fill-opacity', 0.01)
            .attr('stroke', function(d){
                return d.col;
            })
            .attr('stroke-width', function(d){
                return 20*d.sizeMul;
            })
            .attr('stroke-opacity', 0.5)
            .style('stroke-dasharray', '1,3')
            .style('cursor', 'pointer')
            .on('click', connectedNodes)          
            .on("touchstart", connectedNodes)
            .call(d3.drag()
                    .on('start', dragstarted)
                    .on('drag', dragged)
                    .on('end', dragended))
            // .on('mouseout', connectedNodes);

var link = g.append('g')
            .attr('class', 'links')
            .selectAll('line')
            .data(links)
            .enter()
            .append('line')
            .attr('stroke', raspberry)
            .attr('stroke-opacity', 0.5)
            .attr('stroke-width', '2px')
            .style('stroke-dasharray', '10');
```

Now, for what really makes this tick. We first define a tick function to update the positions of the nodes and links at each frame, and then call the force simulation. These functions working together are responsible for the network/force diagram moving around and settling down to their rightful locations on the screen.

And we also add in more functions to drag the nodes, as well and select and deselect nodes and their links.
```
// Function for dragging the nodes
    function dragstarted(d){
    if (!d3.event.active) simulation.alphaTarget(alphaCoeff).restart();
    d.fx = d.x;
    d.dy = d.y;
    }

    function dragged(d){
    d.fx = d3.event.x;
    d.fy = d3.event.y;
    }

    function dragended(d){
    if (!d3.event.active) simulation.alphaTarget(alphaCoeff);
    d.fx = null;
    d.fy = null;
    }

// ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    var toggle = 0;

    function connectedNodes(){
    // The tracker array is used for saving the positions of all the nodes that are connected in the current selection
    tracker = [];
    //  Toggle is needed here as we are using the same function for mouseover and mouseout
    
    if (toggle==0){
        d = d3.select(this).node().__data__;

        node
            .transition()
            .duration(200)
            .ease(d3.easeCubic)
            .style('fill-opacity', function(o){
                return d.index==o.index ? 0.2 : 0.05;
            })
            .style('stroke-opacity', function(o){
                return d.index==o.index ? 0.2 : 0.05;
            });

        nodetext
            .transition()
            .duration(200)
            .ease(d3.easeCubic)
            .style('font-style', function(o){
                return d.index==o.index ? 'bold' : 'normal';
            })
            .style('fill', function(o){
                        return d.index==o.index ? spacecadet : 'grey';
            });

        link
            .transition()
            .duration(200)
            .ease(d3.easeCubic)
            .style('opacity', function(o){
            // If the index of d is equal to either the index of the source or target of the link
                return d.index==o.source.index | d.index==o.target.index ? 0.8 : 0.5;
            })
            .style('stroke', function(o){
                return d.index==o.source.index | d.index==o.target.index ? spacecadet : bluejeans;
            })
            .style('stroke-width', function(o){
                return d.index==o.source.index | d.index==o.target.index ? '2px' : '0.5px';
            });

        toggle = 1;
    } else {

        // Note: Need to use styles instead of attr here for the change to take effect, probably because we used this to access the style
        nodetext.transition().duration(300)
                .style("font-size", textSizeOne)
                .style('font-weight', 'bold')
                .style('fill', spacecadet)
                .style('fill-opacity', 0.5)
                .style('pointer-events', 'none')
                .style('text-anchor', 'middle');	
                
        node.transition().duration(300)
                .style('r', baseRadius)
                .style('fill', bluejeans)
                .style('fill-opacity', 0.01)
                .style('stroke', magenta)
                .style('stroke-width', '20')
                .style('stroke-opacity', 0.5)
                .style('stroke-dasharray', '1,3');

        link.transition().duration(300)
                    .style('stroke', raspberry)
                    .style('stroke-opacity', 0.5)
                    .style('stroke-width', '2px')
                    .style('stroke-dasharray', '10');

            simulation.alphaTarget(0.05);
        toggle = 0;
    }
}
```

And that’s it. 

The actual network/force diagram can be found [here][1]; and the code [here][2].

[1]:	https://playgrdstar.github.io/network_garden_d3/
[2]:	https://github.com/playgrdstar/network_garden_d3

