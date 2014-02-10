---
layout: post
title: Share Jinja2 Templates with the Browser using Nunjucks
description: Sharing templates on the backend with jinja2 and on the client side with nunjucks, all inside flask
date: 2014-02-09 02:23:49 -05:00
tags: "python flask javascript nunjucks jinja2"
---

<div class="alert success">Example Project on <a href="http://github.com/aaboyd/flask-shared-templates">Github</a></div>
## Introduction
After writing a small little web application with [python](http://www.python.org/) and [flask](http://flask.pocoo.org/), I realized I needed templates.  Flask comes with jinja2 support built-in so I opted to take that route and started adding simple templates.  It wasn't long that I realized that in any AJAX heavy application I would need to use the templates both on the front-end (client-side) as well as the back-end (server-side).  After some research I found a great library called [nunjucks](http://jlongster.github.io/nunjucks/).  It's a port of jinja2 to javascript that can be used in the browser and in node.js,

## Setup
The setup for sharing jinja2 templates in the browser is fairly simple.  It should take less than 10 minutes in a brand new project.  Just follow the steps below and you should be up and running in no time.

### Serve Templates to the Client
Flask by default serves things in the static folder.  Add a folder ```templates``` so that nunjucks can load them over HTTP/HTTPS.

### Configure Jinja2
Flask and jinja2 are paired nicely and the configuration is simple.  Below is two different approaches to setting up jinja2.  Flask app's have a ```jinja_loader``` property that can be used to setup the way jinja loads its templates.  Either of the two approaches below will allow you to use the regular flask rendering methods.  ```render_template``` will function just as expected.

#### All Templates Served to the Client
{% highlight python %}
from flask import Flask
from jinja2 import FileSystemLoader
import os

app = Flask(__name__)

base_dir = os.path.dirname(os.path.realpath(__file__))

app.jinja_loader = FileSystemLoader(os.path.join(base_dir, 'static', 'templates'));
{% endhighlight %}

<br />
#### Use Shared Templates and Server-Side Templates
{% highlight python %}
from flask import Flask
from jinja2 import ChoiceLoader, FileSystemLoader
import os

app = Flask(__name__)

base_dir = os.path.dirname(os.path.realpath(__file__))

app.jinja_loader = ChoiceLoader([FileSystemLoader(os.path.join(base_dir, 'templates')),
                                 FileSystemLoader(os.path.join(base_dir, 'static', 'templates'))]);
{% endhighlight %}
<br />

### Configure Nunjucks
Download nunjucks from the [homepage](http://jlongster.github.io/nunjucks/).  There are a few different variants, for starting out I recommend the full version.  In the future you should read about [precompiled templates](http://jlongster.github.io/nunjucks/api.html#recommended-setups), and the recommended setups for production. 

Nunjucks is simple and will run as long as you tell it where to load the templates.  Simply make a call to [```nunjucks.compile```](http://jlongster.github.io/nunjucks/api.html#configure).  In your js file add the following line.

{% highlight js %}
nunjucks.configure('/static/templates');
{% endhighlight %}

After the simple configuration you can make a call to [```nunjucks.render```](http://jlongster.github.io/nunjucks/api.html#render1) to render a template in the browser.

{% highlight js %}
nunjucks.render('hello.html');
{% endhighlight %}
<br />
