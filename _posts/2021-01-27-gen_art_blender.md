---
layout: default
title:  Primer on Generative Art in Blender
description: Primer on Generative Art in Blender
date:   2021-01-28 00:00:00 +0000
permalink: /gen_art_blender/
category: Generative Art
---
## Primer on Generative Art in Blender

I was learning how to use Grasshopper and Python in Rhino3D. Pretty nifty for generating art. Then my trial subscription to Rhino3D ran out. Pity.

Blender seemed to be a rather good open source alternative, with Python as the scripting language. But before learning the Python API in Blender, I needed to get used to the software.

That was probably the hard part. 

Blender’s interface really takes some getting used to, but there are many many good tutorials out there. With that out of the way, picking up the Blender Python API was a breeze.

What I will do in this tutorial is provide a simple primer on -
- How to get started with scripting in Python in Blender
- Creating and manipulating metaballs with a Python script in Blender
- Procedurally generating grids of metaballs to form square, spherical and pyramidal shapes
- Animating the metaballs to shift from shape to shape

**Getting Started With Python Scripting in Blender**
If you are working in Windows, just open Blender normally. A terminal window will open when you launch Blender. However, if you are working on the Mac, one more step will be required, you need to right-click and select ‘Show Package Contents’ and find the blender app in Contents \> MacOS. Only then will the terminal window be available for you to examine error messages to troubleshoot your Python script in Blender.

Next, once Blender is open, choose the Scripting workspace from the dropdown menu directly next to ‘Help’ in the top menu bar. 

The text editor is where you can write your Python script. To run the script, just use the ‘Run Script’ button on the bottom right, or ‘Alt-P’.

**Creating a Metaball using a Python script in Blender**
Creating any object in Blender directly just requires one to ‘Shift-A’ and select what you want created. It’s pretty easy to get started on creating a simple shape through a script even if you know nothing about the API. Just create a shape though the menu, and observe the corresponding code in the Info panel (which is right at the top in the Scripting workspace). For example, move your mouse to the 3D view, ’Shift-A’, and then select Mesh\>Cube to create a cube. You will then see the corresponding code for this in the Info view. 
```
bpy.ops.mesh.primitive_cube_add(radius=1, view_align=False, enter_editmode=False, location=(4.14786, 8.19176, 0.895942), layers=(True, False, False, False, False, False, False, False, False, False, False, False, False, False, False, False, False, False, False, False))
```

You can create a metaball the same way. The metaball is more interesting than a regular shape as metaballs close to each other will actually react to each other. But there is a more efficient way to create metaballs, by creating one main metaball, and then create multiple elements within it.
```
# Selecting the active scene

scene = bpy.context.scene

# Creating the metaball data, instantiating it and attaching it to the active scene.

mball  = bpy.data.metaballs.new('MetaBall')
obj = bpy.data.objects.new('MetaBallObj', mball)
scene.objects.link(obj)

# Setting the resolution of the ball (think of it as how fine you want it to be)

mball.resolution = 0.2
mball.render_resolution = 0.02

arrayballs = [[1,2,3], [2,3,4], [4,5,6]]

for i in range(len(array)):
	coordinate = (arrayballs[i][0], arrayballs[i][1], arrayballs[i][2])
	element = mball.elements.new()
	element.co = coordinate
	element.radius = 0.5
```

The code above should give you a metaball with its elements at the x,y,z locations corresponding to each element in arrayballs.

**Procedurally Generating Grids of Metaballs **
The whole point of scripting is not to do what you could do more easily using the interface, but to be able to procedurally generate stuff. What I have done in this code is to generate arrays of vertices for square, spherical and pyramidal shapes, and then randomly sample a fixed set of points to place the metaball elements at.
```
import bpy
import math
import random 
import numpy as np

def sphere(n,r):
	theta =(math.pi)/n
	phi = (2*math.pi)/n

	vertices = []

	for stack in range(n):
		stackRadius = math.sin(theta*stack) * r
		for slice in range(n):
			x = math.cos(phi*slice) * stackRadius
			y = math.sin(phi*slice) * stackRadius
			z = math.cos(theta*stack) * r
			vertices.append([x,y,z])

	return vertices


s_pts = sphere(20,5)


def pyramid(n, scale):
	vertices = []
	for z in np.arange(-n,n,2):
		for y in np.arange(-z,z,2):
			for x in np.arange(-z,z,2):
				vertices.append([x/scale,y/scale,z/scale])

	return vertices

p_pts = pyramid(40,5)


def square(n, scale):
	vertices = []
	for z in np.arange(-n,n,2):
		for y in np.arange(-n,n,2):
			for x in np.arange(-n,n,2):
				vertices.append([x/scale,y/scale,z/scale])

	return vertices

sq_pts = square(20,5)

def shapePts(array):

	locations = []

	for i in range(400):
		index = random.randint(0,len(array)-1)
		locations.append(array[index])
		
	return locations


scene = bpy.context.scene

mball  = bpy.data.metaballs.new('MetaBall')
obj = bpy.data.objects.new('MetaBallObj', mball)
scene.objects.link(obj)

mball.resolution = 0.2
mball.render_resolution = 0.02


locations = shapePts(s_pts)

for i in range(len(locations)):
	# print(locations[i][0])
	coordinate = (locations[i][0], locations[i][1], locations[i][2])
	element = mball.elements.new()
	element.co = coordinate
	element.radius = 0.5
```

**Animating Metaballs to Shift from Shape to Shape**
The next step demonstrates the power of Python scripting. Animation is a breeze as you can easily set keyframes of different properties for each object in Blender. And your animation is done! No more fussing around with timelines and dopesheets for each object in the scene.
```
currframe = bpy.context.scene.frame_start

bpy.context.scene.frame_set(currframe)

locations = shapePts(p_pts)

for i in range(len(mball.elements)):
	coordinate = (locations[i][0], locations[i][1], locations[i][2])
	mball.elements[i].co = coordinate
	mball.elements[i].keyframe_insert(data_path='co')

bpy.context.scene.frame_set(currframe+60)

locations = shapePts(s_pts)

for i in range(len(mball.elements)):
	coordinate = (locations[i][0], locations[i][1], locations[i][2])
	mball.elements[i].co = coordinate
	mball.elements[i].keyframe_insert(data_path='co')

bpy.context.scene.frame_set(currframe+120)

locations = shapePts(sq_pts)

for i in range(len(mball.elements)):
	coordinate = (locations[i][0], locations[i][1], locations[i][2])
	mball.elements[i].co = coordinate
	mball.elements[i].keyframe_insert(data_path='co')


bpy.context.scene.frame_set(currframe+180)

locations = shapePts(s_pts)

for i in range(len(mball.elements)):
	coordinate = (locations[i][0], locations[i][1], locations[i][2])
	mball.elements[i].co = coordinate
	mball.elements[i].keyframe_insert(data_path='co')

bpy.context.scene.frame_set(currframe+240)

locations = shapePts(p_pts)

for i in range(len(mball.elements)):
	coordinate = (locations[i][0], locations[i][1], locations[i][2])
	mball.elements[i].co = coordinate
	mball.elements[i].keyframe_insert(data_path='co')
```
 
The code above animates the meatball elements moving from a pyramidal to a square and finally to a spherical grid.

And that’s it. The full code is available [here][1].

[1]:	https://github.com/playgrdstar/primer_pythonblender