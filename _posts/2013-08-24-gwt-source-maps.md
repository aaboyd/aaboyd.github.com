---
layout: post
title: Using GWT Super Dev Mode / Source Maps
tags: [gwt, sourcemaps]
description: Quick guide to get up and running with GWT Super Dev Mode and Source Maps in Google Chrome
---

<div class="alert alert-warning" style="font-size:1.25em;font-weight:normal;"><i class="icon-attention"> </i>Updated with various improvements from <a href="https://plus.google.com/u/1/101836723454902363467/posts/bHD9xWqkWKc">Google+ comments</a></div>
One of the biggest gripes I have run into with the [Google Web Toolkit](http://gwtproject.com) is the ability to debug without having to re-compile, clear cache, and be stuck on the Java side of the debugger ( if you use JSNI, you can't really debug it with Eclipse plugin).  Also, the browser plugin often gets broken with new versions of Chrome, and Firefox.  It is not the most "seemless" experience, so I looked for a better option.  Super dev mode works nice in Chrome, which is what I use for GWT development.

GWT has now added support for source maps ( more info on source maps [here](http://www.html5rocks.com/en/tutorials/developertools/sourcemaps/)).  In my eyes it is the future of all [Languages that compile to Javascript](https://github.com/jashkenas/coffee-script/wiki/List-of-languages-that-compile-to-JS).  When supported, it will allow GWT to be debugged in any browser.  I believe Firefox and Chrome currently support it.  I wouldn't be surprised if IE adds support becuase of Microsoft's Typescript.

<div class="alert alert-danger">GWT's Super Dev Mode is only supported in GWT 2.5+.  If you have not upgraded, please upgrade before attempting Super Dev Mode</div>

###Prepare to use Source Maps
To use Source Maps you have to make a few modifications to your ".gwt.xml" file.

{% highlight xml %}
<add-linker name="xsiframe" />
<set-configuration-property name="devModeRedirectEnabled" value="true" />
<collapse-all-properties />
{% endhighlight %}

<span style="text-decoration:line-through">After adding the above, it will require that you recompile.  If you use Eclipse you can do it from the context menu, or however you previously compiled GWT.</span>

###Starting the Super Dev Mode server
To start the super dev mode server you need to execute the gwt-codeserver.jar file.  It is available on [Maven Central](http://mvnrepository.com/), and inside your [GWT SDK](https://google-web-toolkit.googlecode.com/files/gwt-2.5.1.zip).  

Starting the server is pretty simple, it is simply a Java command with a few parameters.

{% highlight bash %}
java -cp "./gwt/sdk/gwt-codeserver.jar:./gwt/sdk/gwt-user.jar:./gwt/sdk/gwt-dev.jar" com.google.gwt.dev.codeserver.CodeServer -src src com.test.MyFirstModule
{% endhighlight %}

The full command line parameters are listed below.

{% highlight bash %}
CodeServer [-bindAddress address] [-port port] [-workDir dir] [-src dir] [module]

where
  -bindAddress  The ip address of the code server. Defaults to 127.0.0.1.
  -port         The port where the code server will run.
  -workDir      The root of the directory tree where the code server will write compiler output. If not supplied, a temporary directory will be used.
  -src          A directory containing GWT source to be prepended to the classpath for compiling.
and
  module        The GWT modules that the code server should compile. (Example: com.example.MyApp)
{% endhighlight %}

After running the CodeServer application you will get a  message with the URL that is serving your source.  Simply visit that URL, and you should see 3 pretty simple instructions.  Make sure you do 1., 2 is a bit more tricky then they explain.

### <span style="text-decoration:line-through">Modifying and</span> Serving your HTML file
<span style="text-decoration:line-through">The HTML file that hosts your application, will have to be modified.  In Step 2 of the Source Maps server instructions, it tells you to visit a page that uses one of the modules.  To me this was a bit misleading, because at this point, nothing is actually serving your HTML file.

First, modify your Host HTML page to redirect the
{% highlight xml %}
<script src="module/module.nocache.js"></script>
{% endhighlight %} to your Source Maps server :

{% highlight xml %}<script src="http://localhost:9876/module/module.nocache.js"></script>{% endhighlight %}.

After you have redirected your page to load the correct ".js" files,</span>

You need to serve the HTML file somehow.  The simplest and quickest way I found to serve the HTMl files, was to use python ( I am aware this is probably not the best and most reliable method, but it works ).  Navigate to the root directory of your HTMl file, and execute :

{% highlight bash %}
python -m SimpleHTTPServer
{% endhighlight %}


###Enable Source Maps in Chrome
	1. Open chrome dev tools
    2. Open Settings (gear in bottom right of dev tools)
    3. In "Source" section, enable source maps

If you notice you can navigate and step through both the Java code and jsni methods (something that was never available before).  Also, recompiling the module is as easy as clicking "Dev Mode On" again and compiling the new module.  Compilations are much quicker than full GWT compiles.  The incremental builds save a lot of time.

Even if the Eclipse/IntelliJ plugin paired with chrome's plugin are good enough for you, source maps still have some advantages.  Source maps appear to be a new techonology that will be used more and more, so getting familiar with using it now should benefit you in the future.
