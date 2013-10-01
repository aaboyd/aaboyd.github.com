---
layout: post
title: "Object-Oriented Javascript … A First Attempt"
tags : [javascript]
description : Learning to create and use JavaScript coming from a C/C++, Objective-C and Java background.
---
## Non-JavaScript Background
I come from  a C/C++ background which later became Objective-C and Java (iOS and Android).  I haven't done a ton of web work, but most of the web stuff i have done I used the [Google Web Toolkit](http://gwtproject.com), which provides Java to JavaScript transcompilation.  I have never gotten into the nitty gritty of the JavaScript language, so I am writing this as a cry for help, PLEASE! help me become a better JavaScript developer.

<div class="alert warning"><i style="display:inline-block" class="icon-star"> </i><span style="display:inline-block">PLEASE READ</span> <i style="display:inline-block" class="icon-star"> </i> <br /> I am no JavaScript expert, nor do I claim to be one.  I simply was tasked with writing some JavaScript at work, and I would like to be critiqued on my work.  Any / ALL comments, concerns, and pointers are welcome.  I hope the community can help me work out the kinks in my first days with Javascript</div>

# Javascript Objects 

After researching a bit on how to define JavaScript "classes", I have discovered what I believe to be a simple enough approach.  I have made a few iterations on my approach, starting with the below definition which is my first attempt at creating and defining a JavaScript object definition / prototype, usually referred to as a class in many other languages. 

## Object Definition 

{% highlight js %}
var Fruit = {
  
  price: null,
  name: null,

  getPrice: function() {
    return this.price
  },
  
  print: function() {
    return '[' + this.name + '] - $' + this.price
  },
  
  isEdible: function() {
    throw {
      name: "Unimplemented",
      message: "isEdible is not implemented in the base 'Fruit'"
    }
  }
};
{% endhighlight %}

## Creating an Instance 

Creating an instance of a JavaScript prototype is very familiar.  The ```new``` keyword exists and it does exactly what you would expect. 

{% highlight js %}
var Apple = new Fruit 
Apple.price = 0.00 
Apple.name = "apple" 
{% endhighlight %} 

### Creation looks wrong ?
What looks a little weird is the missing ```()``` after ```Fruit```.  This is a bit unnatural to me.  To get the parenthesis I was looking for I had to use a ```function()``` to wrap up my current functionality.  When doing this, the function has a little bit of a different syntax, but for the most part things are the same.  The  new definition introduces the ```this``` keyword, which is used to reference the current object.  Same as in many other languages.

{% highlight js %}
var Fruit = function () {

  this.price = null;
  this.name = null;
  
  this.getPrice = function () {
    return this.price
  };

  this.print = function () {
    return '[' + this.name + '] - $' + this.price
  };

  this.isEdible = function () {
    throw {
      name: "Unimplemented",
      message: "isEdible is not implemented in the base 'Fruit'"
    }
  }
};
{% endhighlight %} 
 
## Constructor Arguments
This gives me the initialisation I are looking for.  Also, I can conveniently pass in variables to the constructor, so with mild modification I can pass in price and name instead of using three lines to create the desired object. 

{% highlight js %}
var Fruit = function(p, n) {

  this.price = p;
  this.name = n;
  
  ...
  
}
{% endhighlight %}  

## More familiar Instantiation
Now I can easily create an object and it will look almost exactly like it would in another language like Java. 

{% highlight js %}
var Apple = new Fruit(1.00, "apple") 
{% endhighlight %} 

In addition to the better looking constructor there is also something that I picked up while reading other blog posts and stackoverflow.  Your JavaScript should be isolated from the global namespace.  There is no need to 'dirty' the global namespace if it is not needed.  In the example above anything declared as a ```var``` inside the function will actually be attached to the global namespace.  My ```Fruit``` example does not do this, but often times other objects will.  To avoid this, you use a self executing function object. 

{% highlight js %}
var Fruit = (function (p, n) {
  this.price = p;
  this.name = n;

  ...

})() 
{% endhighlight %} 

## Make it inheritable
I have created a JavaScript object definition, isolated it from the global namespace, added parameters to the constructor, and easily instantiated it.  After figuring out the different methods to define 'objects' in JavaScript my next question was how to do inheritance.  This brought me to using an object prototype to define functions once and have them be used multiple times by multiple objects.  If you don't know what a prototype is you should poke around on the internet and read some articles.  It is really just a JavaScript object that defines functionality on an object so it can easily be shared between instances.  Below is my example transformed to use an object prototype.  First, create a variable that will represent our classes constructor, then create the functions inside that classes prototype.  Like before, syntax has changed a bit, but the end result is really the same. 

{% highlight js %}
var Fruit = (function () {

  var clazz = function (p, n) {
    this.name = n
    this.price = p
  };

  clazz.prototype = {
    getPrice: function () {
      return this.price
    },

    print: function () {
      return '[' + this.name + '] - $' + this.price
    },

    isEdible: function () {
      throw {
        name: "Unimplemented",
        message: "isEdible is not implemented in the base 'Fruit'"
      }
    }
  }

  return clazz;
})();
{% endhighlight %}

## Inherit It
Everyone is well aware that there is more than one type of fruit ( or maybe I just spilled the beans ).  To incorporate this into my example, I want to keep the functionality that i have for ```Fruit```, but ```Apple``` has a fixed name and price.  Below is a fairly easy example of inheriting an Object's prototype.

{% highlight js %}
var Apple = (function () {

  var clazz = function () {
    Fruit.call(this, 1.00, "apple")
  };

  clazz.prototype = Fruit.prototype

  clazz.prototype.isEdible = function () {
    return true;
  }

  return clazz;
})();
{% endhighlight %}

The code is pretty straight forward.  2 things to note.
1. The constructor of Apple takes no arguments.  To call the constructor of the base object you use the ```call``` method to execute a function with a specified ```this``` variable.  Also, allows for passing in other variables as you can see above.  The constructor for ```Apple``` should simply be ```new Apple()```
2. To take advantage of the functions defined in the ```Fruit``` prototype I simply set our  new object's prototype equal to the base prototype.  All the methods in ```Fruit``` are now available in our new object.

I've managed to build a simple standard for myself which really helps with learning JavaScript.   It isn't perfect and may diminish some of the power behind JavaScript, but coming to these examples has made my JavaScript more managable / readable.


