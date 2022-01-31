---
layout: default
title:  (Almost) The Simplest Server Ever
description: (Almost) The Simplest Server Ever
date:   2021-01-31 00:00:02 +0000
permalink: /simple_server/
category: Data Engineering
---
## (Almost) The Simplest Server Ever

What good is any script you have written if all you can do is send it round as a .py file or as a Jupyter notebook? 

What we really want to do is to be able to deploy that script in some way online for all to access.

The first step on that path would be to learn a web framework. There are a whole range of options, from Express in Javascript to Flask or Django in Python. Flask is probably one of the easiest to pick up. So that’s where we will start. 

I wanted to title this as the simplest server ever. But that would mean a server which prints its outputs to the terminal. And that won’t be much fun. 

So this is (almost) the simplest server ever, because we will include a simple set of templates, and hence be able to see webpages being served by the server. 

I will, in the next related tutorial, then show to integrate some useful functions so we can use the server to serve something useful.

First off, install Flask. If you don’t know how to install packages in Python, do a search on ‘pip install’. So for flask all we need to do in any terminal is do type this in.
```
pip install flask
```

Next, we write a Python script, named _run.py_. 

**Main script to run server**
So for the run.py script, we first import the libraries we need - the main Flask class, as well as the `render_template` function.
```
from flask import Flask
from flask import render_template
```

We then instantiate a Flask object.
```
app = Flask(__name__)
```

The next two parts are almost identical. What they do is to set a route (which basically means the relative address/path of the webpage). And when the user goes to that page, a webpage is rendered based on a template, and the `title` and `bodytext` are passed to the page.

That may be confusing. So let me explain a little bit more. We are basically saying that when the user goes to the domain (e.g. domain.com/), he will see a webpage rendered using the HTML template `base.html` and the values assigned to `title` and `bodytext` will be passed to the template. You will see how we read in `title` and `bodytext` later when we do the templates. The parts right at the top `@app.route("/")` are called decorators. We shall not go into the details on these now, but will cover them another time.
```
@app.route("/")
def main():
    return render_template('base.html', 
                        title='This is the title', 
                        bodytext='This is the body text')

@app.route("/extended")
def extended():
    return render_template('extension.html', 
                        title_extended='This is the title of another page', 
                        bodytext_extended='This is the body text of another page ')
```

And finally, we run the app. The first line is just to check that the run.py file is the main file being run (i.e. this file is not being called by another file). We can set the host and port we want to access the webpage from here too.
```
if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True, port=8000)
```

**Templates**
Now we move to the templates. `base.html` is the base template, and `extension.html` extends (or is based on) the base template.

For `base.html` we basically do the normal HTML stuff, which I have covered before, but with two key differences. We add in stuff between the mustaches `{}`. Basically what we are doing is to either - 
- Allow the blocks enclosed within `{% %}`to be replaced when we extend this base template; and
- Pass in values assigned to `{{title}}` or `{{bodytext}}` which we declared earlier when we set up the server.
And that’s it. These are the only differences between a template and the usual static HTML pages.

```
<title>
    {% block title %}Simplest Server Ever{% endblock %}
</title>

...

{% block content %}

<div>
    <h4 id='title'>{{title}}</h4>
</div>

<div>
    <h6>{{bodytext}}</h6>
</div>

{% endblock %}
```

The great thing about such templates is that we don’t have to keep re-writing everything. We can build on the base template by inserting this line in `extension.html`.
```
{% extends 'base.html' %}
```

We can then render everything in `base.html`, and replace anything within `{% block content %} ` and `{% endblock %}` with the corresponding block in `extension.html`.
```
{% block content %}

<div>
    <h4 id='title'>{{title_extended}}</h4>
</div>

<div>
    <h6>{{bodytext_extended}}</h6>
</div>

{% endblock %}
```

And that’s it. Now all you have to do is to type `python run.py` in your terminal and you will see the webpages being served on -

- [http://0.0.0.0:8000/][1]
- [http://0.0.0.0:8000/extended][2]

The code for this tutorial is available [here][3]

[1]:	http://0.0.0.0:8000/extended
[2]:	http://0.0.0.0:8000/extended
[3]:	https://github.com/playgrdstar/almost_simplest_server "Almost the Simplest Server Ever"