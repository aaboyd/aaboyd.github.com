---
layout: post
title: "Building this blog"
tags : [jekyll, pretzel, chocolatey, github]
---
##Exploration of technologies
###Blog Engine
I work on Mac OS X at work, but often times find myself learning new technologies / "messing around" on a Windows 8 machine.  So, I looked at a C# blog engine.  Didn't really find anything that was really simple to host and easy to modify (I am lazy, anything that required significant work, I gave up on).

After reading a few other posts about setting up blogs, I read about static-site generatos.  After poking around a bit, [Jekyll](http://jekyllrb.com/) seems to be the most popular, probably because of its origins and support on github.  After finding Jekyll, I ran across [Pretzel](https://github.com/Code52/pretzel), it's C# equivalent.  I liked this because, I hope to eventually convert this blog to C# if possible.

###Look and Feel
I was looking for something that was simple, easy-to-use, and good-looking.  The obvious candidate [Twitter Bootstrap](http://twitter.github.io/bootstrap/) looked good, but I wanted to be different.  Many people have used Twitter Bootstrap and had great success, I wanted to try to be a bit original.  I found and am still using [Gumby](http://gumbyframework.com/).  The two frameworks are similar in many ways, each having advantages / disadvantages, but that is a discussion for another day.

###Deployment
The easiest quickest deployment option I have found was github pages, and that is where it is hosted right now.  Maybe in the future I will mess around with different deployment options.  The build / deployment engineer inside me always looks for fancy ways to build projects and deploy them to production

####Testing Deployment
After a few commits, and slimming down the blog to it's bare necessities (I am a minimalist, so Jekyll Bootstrap was a good start but overkill), I was getting failures on github pages.  Because I am not a ruby guy, I needed to install and run jekyll on Windows (more difficult than Mac OS X or Linux).  I use [Chocolatey](http://chocolatey.org) which everyone should install :).  Here is a quick gist of what needs to be executed to get [Github Pages](http://pages.github.com) ready.
<script src="https://gist.github.com/aaboyd/5696136.js"> </script>

###Disqus

###Google Analytics

Getting the exact test environemnt used for github pages, made deployments easy to test.  Fixed a few layouts, fixed a few posts, and boom! site is now up and running.

##What to write about?
* Do I have anything interesting to write?
* Will people listen?
* Will I prove that I am really dumb?

Those are a few of the questions preventing me from starting a blog.  It hasn't been much work getting things up and running, but content is everything.  If nothing is good here, than no one will come.  I plan to keep this blog underground for a bit, and letting it into the wild after a few posts have been generated.  See if anything catches users eyes.
