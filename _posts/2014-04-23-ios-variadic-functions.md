---
layout: post
title: "Variadic Objective-C Functions ( variable function parameters )"
tags: [ios, objective-c]
description: Objective-C Functions that accept a variable number of parameters / arguments
comments: true
---

## What are variadic functions?
Well, to be honest, I wasn't exactly sure the name of these kinds of functions for years after I started programming.  Just knowing the terminology will make your life a lot easier when googling for this same kind of thing in different programming languages.

>In computer programming, a variadic function is a function of indefinite arity, i.e., one which accepts a variable number of arguments. Support for variadic functions differs widely among programming languages. (stolen from [Wikipedia](http://en.wikipedia.org/wiki/Variadic_function))

In the simplest of terms, it is a function that takes any number of arguments vs a function that takes a defined number of arguments.

## iOS Library usage
There are multiple places that you run into variadic functions when doing iOS development.  They are fairly simple to understand and use.  The variable parameter can contain a comma seperated list of values and must be `Nil` terminated.  Example below demonstratest creating a `UIAlertView` without any syntax

{% highlight objc %}
UIAlertView* alert = [[UIAlertView alloc] initWithTitle:@"Title"
                                                message:@"Message"
                                               delegate:nil
                                      cancelButtonTitle:@"Cancel"
                                      otherButtonTitles:@"OK", @"Ignore", nil];
{% endhighlight %}

## Implementing your own Variadic function
Using these functions is simpler than creating your own function that accepts a variable parameter.  Much of the syntax and usage is inherited from C, so it may seem a little bit foreign if Objective-C is all you know.  Example below demonstrates sorting any number of `NSString`'s.

{% highlight objc %}
-(NSArray*) sort:(NSString*) values, ...
{
    NSMutableArray* sortableItems = [[NSMutableArray alloc] init];

    if( values == nil )
        return sortableItems;

    va_list args;
    va_start(args, values);

    NSString* str = values;
    do
    {
        [sortableItems addObject:str];
    }
    while( (str = va_arg(args,NSString*)) );

    va_end(args);

    return [sortableItems sortedArrayUsingSelector:@selector(caseInsensitiveCompare:)];
}
{% endhighlight %}

### Explanation
The first thing to notice is the method definition.  This is just syntax, but the `...` signifies that this is a variable argument.  The second thing is the `nil` check, values will always be a pointer to the first element of the variable argument, if the first element is `nil` we simply return an empty array.

The c-style stuff comes into play when you reach the `va_list` line.  It is pretty simple, but you create a `va_list` and start reading from that list using `va_start`.  I used a `do , while` loop because the `va_arg` function will move the pointer, and I wanted access to the first item.  After you are done, of course, call `va_end`.

That's all there is to making variadic functions in iOS
