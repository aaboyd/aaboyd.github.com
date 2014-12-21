---
layout: post
title: "Know Your Tools, Don't Embarass Yourself"
tagline: Diagnosing files ignored by git
tags : [git, career]
description : Way too much information on how I embarassed myself and I have no one to blame except me.
---

**TL;DR**

<div class="alert alert-success">Your tools are powerful, your probably not using them to their potential.  Learn those tools and you will save yourself time later.</div>

## What happened?

About a week or two ago, I started running into problems with my QA team and had a terrible time reproducing some issues in our iOS app.  At first I checked for inconsistencies with the build server.  Everything looked ok, so I started digging a little deeper.  After far too long I came to the conclusion that the only logical reason I could be out of sync was becuase of some files that I wasn't checking in.  Below I outline what I did, and how I could have done it better, but that is not the point of this post.

## Why did it happen?

I have always used [SourceTree](http://sourcetreeapp.com) for my [git](http://www.git-scm.com/) and [mercurial](http://mercurial.selenic.com/) repos.  I think it's a great tool and I will continue to use it, but I have neglected my knowledge of git and mercurial.  I haven't learned to use my tools efficiently.  I ended up making a silly mistake that made me lose hours of time.  I've recently begun splitting time between the command and SourceTree and I am constantly surprised by the power of git.

I use a ton of different tools daily and I've vowed to dig a little deeper into the power and functionality of those tools.  Between Eclipse, Android Studio, Google App Engine, XCode, PyCharm, etc.  I have my work cut out for me.

## Details of my situation

### The easy way

#### Find what files are being ignored 

{% highlight bash %}
git clean -x --dry-run
{% endhighlight %}

#### Why is git ignoring this file
{% highlight bash %}
𝝺 git check-ignore /path/to/file -v
.gitignore_global:2:/path/to/file  /path/to/file
{% endhighlight %}

#### Look at the gitignore rule

{% highlight bash %}
𝝺 cat ~/.gitignore_global
.DS_Store
*.js
{% endhighlight %}

Oh no!  I don't want to ignore all javascript files.  This made it in here somehow, I'm not exactly sure how.  Removing the line and everything is back to normal.

### The way I did it == terribly inefficient!

<div class="alert alert-info">This is probably the LEAST efficient way to find / diagnose issues like this, but my command line skills were enough to get me to the finish line.  That's good I guess</div>

#### Re-clone the git repo, completely
{% highlight bash %}
git clone {GIT_URL} test-repo
{% endhighlight %}

#### Diff the directories
{% highlight bash %}
diff -r /path/to/old/repo /path/to/new/clone
{% endhighlight %}

Finding the suspicious files only took me about a second.  The output from the diff was full of stuff, but not too much was important.

#### Look for all the .gitignore files and see if they are ignoring the file
{% highlight bash %}
find . | grep ".gitignore" # there was only one in my case
cat .gitignore
{% endhighlight %}

#### Look at the git config to find out where the global gitignore is
{% highlight bash %}
𝝺 git config -l
...
core.excludesfile=/Users/alex/.gitignore_global
...
{% endhighlight %}

#### Look at the gitignore rule

{% highlight bash %}
𝝺 cat ~/.gitignore_global
.DS_Store
# remove the line below
*.js   
{% endhighlight %}

## The different gitignore files

### The .gitignore that everyone uses

Including a ```.gitignore``` in the root of your git repository is pretty standard practice.  Creating / maintaining the correct configuration isn't too difficult, but there are many different tools that can aid in the process.  Many of the GUI tools out there provide functionality to add / remove to a gitignore file, I personally use [Sourcetree](http://www.sourcetreeapp.com/) and [Gitignorer](https://github.com/zachlatta/gitignorer).

### Ignoring files that aren't shared with others

From the root of your repository, you can create / edit a ```.git/info/exclude``` file.  This is useful if you want to add some files to the repository for your local setup but want to avoid making the ```.gitignore``` dirty.

### Ignoring only beneath a particular directory

You can add a ```.gitignore``` file inside any child directory and it will take only apply to that directory and it's children.  This is not used as often but sometimes can be simpler than having an extremely long .gitignore.

### Global Ignore

A global git ignore can be configured, just execute the following

{% highlight bash %}
git config --global core.excludesfile ~/.gitignore_global
{% endhighlight %}
