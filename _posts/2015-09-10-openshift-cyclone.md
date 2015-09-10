---
layout: post
title: "Twisted Python / Cyclone, 'Hello, World!' application, on Openshift"
tags : [python, openshift]
description : Running a simple http://cyclone.io application on (https://www.openshift.com/
---

<div class="alert alert-info">I am working on Mac OS X, some of the commands in this tutorial may not be identical on your OS</div>

# Prerequisites
* [Git](http://www.git-scm.com/)
* [Ruby](https://www.ruby-lang.org/en/)
* [OpenShift Online Accont](https://openshift.redhat.com)
* [OpenShift Command Line Tools](https://developers.openshift.com/en/getting-started-overview.html)
* [Python](https://www.python.org/)
* [Pip](https://pypi.python.org/pypi/pip/)
* [Twisted](https://twistedmatrix.com/trac/)
* [Cyclone](http://cyclone.io)

<div class="alert alert-danger">Be sure to complete the openshift setup as described [here](https://developers.openshift.com/en/getting-started-overview.html)</div>

# Create Application (Gear) on OpenShift Cloud

{% highlight bash %}
𝝺 rhc create-app cyclonetest python-2.7
{% endhighlight %}

<div class="alert alert-warning">There may be a few prompts about ssh keys, don't be alarmed</div>

# Install Application Dependencies

{% highlight bash %}
𝝺 cd /path/to/application
𝝺 echo "cyclone==1.1" > requirements.txt
𝝺 pip install -r ./requirements.txt
{% endhighlight %}

# Create "Hello, World!" Application

{% highlight bash %}
𝝺 cyclone app -n > app.py
{% endhighlight %}

## Verify Application Works

### Start Application
{% highlight bash %}
𝝺 cyclone run --app=app.py
{% endhighlight %}

### Test the app

#### Using [cURL](http://curl.haxx.se/)
{% highlight bash %}
𝝺 curl http://localhost
Hello, world
{% endhighlight %} 

#### Using [HTTPie](http://httpie.org)
{% highlight bash %}
𝝺 http --print=HBhb http://localhost:8888
GET / HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Connection: keep-alive
Host: localhost:8888
User-Agent: HTTPie/0.9.2



HTTP/1.1 200 OK
Content-Length: 12
Content-Type: text/html; charset=UTF-8
Date: Thu, 10 Sep 2015 02:54:23 GMT
Etag: "e02aa1b106d5c7c6a98def2b13005d5b84fd8dc8"
Server: cyclone/1.1

Hello, world

{% endhighlight %}

### Modify App to Incorporate Openshift Environment

#### Add Import Statements

{% highlight python %}
import os, sys
from twisted.python import log
from twisted.internet import reactor
{% endhighlight %}

#### Add __main__

{% highlight python %}
if __name__ == "__main__":
    log.startLogging(sys.stdout)
    port = int(os.getenv('OPENSHIFT_PYTHON_PORT', 8888))
    host = os.getenv('OPENSHIFT_PYTHON_IP', "0.0.0.0")
    reactor.listenTCP(port, Application(), interface=host)
    reactor.run()
{% endhighlight %}

#### Full Application

{% highlight python %}
import os, sys
from twisted.python import log
from twisted.internet import reactor

import cyclone.web

class MainHandler(cyclone.web.RequestHandler):
    def get(self):
        self.write("Hello, world")


class Application(cyclone.web.Application):
    def __init__(self):
        handlers = [
            (r"/", MainHandler),
        ]

        settings = dict(
            xheaders=False,
            static_path="./static",
            templates_path="./templates",
        )

        cyclone.web.Application.__init__(self, handlers, **settings)

if __name__ == "__main__":
    log.startLogging(sys.stdout)
    port = int(os.getenv('OPENSHIFT_PYTHON_PORT', 8888))
    host = os.getenv('OPENSHIFT_PYTHON_IP', "0.0.0.0")
    reactor.listenTCP(port, Application(), interface=host)
    reactor.run()
{% endhighlight %}

# Download .gitignore for python projects

{% highlight bash %}
curl https://raw.githubusercontent.com/github/gitignore/master/Python.gitignore > .gitignore
{% endhighlight %}

# Commit and Push

{% highlight bash %}
𝝺 git add -A
𝝺 git commit -am "Adding cyclone hello world"
𝝺 git push
{% endhighlight %}

# Enjoy!

{% highlight bash %}
𝝺 rhc show-app cyclonetest
testcyclone @ http://cyclonetest-alexboyd.rhcloud.com/
{% endhighlight %}

{% highlight bash %}
𝝺 curl http://cyclonetest-alexboyd.rhcloud.com/
Hello, world
{% endhighlight %}