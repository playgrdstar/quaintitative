---
layout: default
title:  Data Analytics with Blender
description: Data Analytics in Blender - Pandas and Quandl in Blender
date:   2021-01-29 00:00:00 +0000
permalink: /data_blender/
category: Visualisation
---
## Data Analytics in Blender - Pandas and Quandl in Blender

Once I knew that I could use Python to script in Blender, an obvious question that came to mind was whether I could use it for data analytics and visualisation.

And the answer is of course, yes!

But before I begin showing how one could visualise data within the Blender environment, an important first step is how one could use key libraries needed for data analytics in Blender. 

There are a number of ways this could be accomplished, but in this tutorial, I will just share my preferred way of importing the Pandas and Quandl libraries within the Blender Python environment.

The Python that runs within Blender is not the one that you usually use on your computer. It sits in a different location, and looks for installed libraries in different folders. 

To see the paths that Blender’s Python checks, simply type the following in the Python console.
```
import sys
print(sys.path)  
```

You could get pip installed in one of these paths, and be able to install libraries directly within Blender’s Python. But I rather keep these clean. It’s simpler to start a new environment with virtualenv, install the libraries you want within that, and get Blender to look for libraries at in that virtualenv instance.

You need to first make sure that the version of Python on Blender is the same as that in the virtualenv instance. Check that by typing the following in the Python console.
```
import sys
print(sys.version)
```

Once you know that (it should be 3.5 on Blender 2.79), start a virtualenv with the same version of Python.
```
virtualenv -p python3 venv
```

Activate it.
```
source venv/bin/activate
```

Install what you need using pip.
```
pip install pandas
pip install quandl
```

Now, check where the location of the libraries are. Look for the one with ‘site-packages’
```
echo "import sys; print(sys.path)" | python 
```

Go into the Python console in Blender and add this path.
```
import sys
sys.path.append('/Users/<user-name>/<path>/<to>/venv/lib/python3.5/site-packages')
```

And that’s it. You can now use pandas to munge data; or quandl to fetch data from within Blender! Do note that the path you just appended is not persistent. When you restart Blender, you will need to append the path again.



