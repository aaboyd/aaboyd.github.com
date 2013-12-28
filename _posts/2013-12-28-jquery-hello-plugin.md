---
layout: post
title: "jQuery Hello World Plugin with Structure"
tags : [javascript, jquery]
description : jQuery "Hello World" plugin with some structure
---
## jQuery, a necessary language extension
My first brushes with Javascript were through [GWT (Google Web Toolkit)](http://www.gwtproject.org/).  I was unaware of how closely related and intermingled JavaScript and [jQuery](http://jquery.com/) have become.  Many job postings list jQuery in the section right along with other languages.  Even if you decide not to use jQuery, or decide to use a lighter alternative like [Zepto.js](http://zeptojs.com/), the principals for plugins and overall architecture are lessons that can be applied elsewhere.

## Motivation
Recently, having started a new job and picking up a lot of frontend work, I have been working with a lot of jQuery and jQuery plugins.  3rd party libraries, other devs, and I all have our own styles.  So I set out to provide some structure and organization into my life.  Here is my attempt at writing a jQuery "Hello World" plugin to use as my template for future jQuery plugins. 

## Let the code do the talking
All the code can be found on github at [https://github.com/aaboyd/jquery-hello-plugin](https://github.com/aaboyd/jquery-hello-plugin).  It is commented a lot to help explain the reasoning and logic behind the structure.

### Desired API
When I set out to make this template, I had an api in mind.  It looked a little something like:
{% highlight js %}
var helloPlugin = $("#elem-id").hello({
	name: 'Plugin',
});

helloPlugin.sayHello('{Your Name}');
{% endhighlight %}
The initialization is pretty standard jQuery stuff, I'm not sure how much that could actually change.  Return value can be different depending on the library or framework.  I like to have a "handle" to the plugin which will allow me to call methods on the plugin later.  Some do callbacks and events for all interactions, but I find it handy to get a "handle".

### A Single Plugin Object
Objects are a tricky thing in JavaScript.  I learned a few things about JavaScript objects and I wrote about them a little while back [Object-Oriented Javascript … A First Attempt](http://aaboyd.github.io/2013/10/oop-javascript-first-try/).  To apply that concept to jQuery development, I like to have one single object definition that gets instantiated when the plugin is called.

{% highlight js %}
// Plugin constructor
var Hello = function ($elem, options) {
  this._init();
}

$.extend(Hello.prototype, {
  _init : function () {
    ...
  },
    
  sayHello : function (name) {
    ...
  },
});
{% endhighlight %}

### Options
Almost all jQuery plugins will have some sort of options.  If your writing a plugin that doesn't have options, you should look to extend it and make it more generic and applicable for other scenarios.

{% highlight js %}
var defaultOptions = {
  name: "World"
};

var Hello = function ($elem, options) {

  this.settings = $.extend({}, defaultOptions, options);
  
  this._init();
}
{% endhighlight %}

```defaultOptions``` are defined outside of ```Hello``` because they will never change.  To merge the options passed in with the plugins constructor jQuery has a convenient [$.extend](http://api.jquery.com/jQuery.extend/) method.  Now inside ```Hello``` and all of the ```Hello.prototype``` methods, this.settings will be accessible.

### Adding to jQuery
Adding to the ```$``` or ```jQuery``` variable is very simple.  The initial construction of an object contains some logic and guards against instantiating twice, and also provides a mechanism for what is returned.

{% highlight js %}
$.fn[pluginName] = function (methodOrOptions) {
  var plugins = [];

  this.each(function () {
    var instance = $(this).data("plugin-" + pluginName);
    if (!instance) {
      instance = new Hello($(this), methodOrOptions)
      $(this).data("plugin-" + pluginName, instance);
    }
    plugins.push(instance);
  });

  return (plugins.length === 1) ? plugins[0] : plugins;
};
{% endhighlight %}

```pluginName``` is a variable defined at the top of the js file, purely for style reasons, I think it is nice to see the name at the beginning of the file.  This function get's called with ```this``` being the element or elements that were selected with [jQuery's Selector's](http://api.jquery.com/category/selectors/).  The ```this.each``` will iterate through all the elements and then do the initialization process.  Try to get a handle of the plugin that is attached to the DOM element, if it does not exist create it and attach it to the DOM.  This will prevent re-initialization of the plugin.  Also, add the instance variable to the list of plugins.  The return statement is just a convenience. If a plugin was initiailized on an individual element it will return the plugin, if not it will return the array of plugins.

### Adding AMD module support
[Require.js](http://requirejs.org/) has become a powerful way to manage JavaScript modules.  To allow your plugin to be used with Require.js the plugin must be wrapped in a ```define``` function.  Wrapping all the previous code inside the following block will give you both require.js compatibility and standard JS script tag support.

{% highlight js %}
(function (factory) {
  if (typeof define === 'function' && define.amd) {
    define(['jquery'], factory);
  } else {
	factory($);
  }
}(function ($) {
  ...
}));
{% endhighlight %}

## Resources
I read a lot of different resources to come up with the following template.  I have also iterated a few times on what I want / need.  I encourage you to research and find your own template to work from.  I read the following (and even though I didn't use everything or anything from some of them), they are all good resources.

* [jQuery Plugins](http://learn.jquery.com/plugins/)
* [jQuery Boilerplate](http://jqueryboilerplate.com/)
* [Essential jQuery Plugin Patterns](http://coding.smashingmagazine.com/2011/10/11/essential-jquery-plugin-patterns/)
* [jQuery Plugin with methods](http://stackoverflow.com/questions/1117086/how-to-create-a-jquery-plugin-with-methods)
* [jQuery Plugin loaded with Require.js](http://stackoverflow.com/questions/10918063/how-to-make-a-jquery-plugin-loadable-with-requirejs)


