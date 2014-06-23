---
layout: post
title: "Where is my Android Application, Foreground or Background?"
tags : [java, android]
description : One way to detect if your android application is in the foreground or background.
---

Detecting if an Android application is in the foreground or the background seems like it might be obvious, but it actually isn't as easy of a problem as I thought.

Android has come a long way from it's 2.X days, but there are still a lot of users out there looking for good apps that have 2.3.X devices ( I have given up on 2.2 ).  So, I was looking for the best way to see if an app is in the foreground or the background.  There are few different approaches discussed on this stackoverflow post : [Android: Is Application running in background?](http://stackoverflow.com/questions/3667022/android-is-application-running-in-background/5862048).  The solution I came up with is mostly adapted from that post, but it involves implementing reference counting in the activities manually.

[ActivityLifecycleCallbacks](http://developer.android.com/reference/android/app/Application.ActivityLifecycleCallbacks.html) are all that one really needs to make their application aware of when an activity lifecycle event happens.  So, mimmicking it's behaviour is what I set out to do.

The implementation will involve two classes.  The GlobalApplication object derives from [Application](http://developer.android.com/reference/android/app/Application.html).  The other class needs to be the base of all your activity classes.

The GlobalApplication is fairly simple.  It implements two methods to be called when an activity is resumed and when an activity is paused.

{% highlight java %}
/**
 * TODO : After update to API level 14 (Android 4.0),
 * We should implement Application.ActivityLifecycleCallbacks
 */
public class GlobalApplication extends android.app.Application
{
    private boolean inForeground = true;
    private int resumed = 0;
    private int paused = 0;

    public void onActivityResumed( Activity activity )
    {
        ++resumed;

        if( !inForeground )
        {
            // Don't check for foreground or background right away
            // finishing an activity and starting a new one will trigger to many
            // foreground <---> background switches
            //
            // In half a second call foregroundOrBackground
        }
    }

    public void onActivityPaused( Activity activity )
    {
        ++paused;

        if( inForeground )
        {
            // Don't check for foreground or background right away
            // finishing an activity and starting a new one will trigger to many
            // foreground <---> background switches
            //
            // In half a second call foregroundOrBackground
        }
    }

    public void foregroundOrBackground()
    {
        if( paused >= resumed && inForeground )
        {
            inForeground = false;
        }
        else if( resumed > paused && !inForeground )
        {
            inForeground = true;
        }
    }
}
{% endhighlight %}

In the above example a some code is left out.  I use a custom event bus system that allows me to queue events for a future time.  This portion can be implemented in many different ways, so I left it up for interpretation.  Due to the lifecycle of android activities, when you call finish then startActivity you can get the events in the opposite order you want.  I use this to stop and start tasks that run without the user being aware, so half a second delay is not a problem in my case, for others it may be.

The base class for all your activities needs to override the onResume and onPause methods and call them on the GlobalApplication.

{% highlight java %}
public class BaseActivity extends android.app.Activity
{
    private GlobalApplication globalApplication;

    @Override
    protected void onCreate()
    {
        globalApplication = (GlobalApplication) getApplication();
    }

    @Override
    protected void onResume()
    {
        super.onResume();
        globalApplication.onActivityResumed(this);
    }

    @Override
    protected void onPause()
    {
        super.onPause();
        globalApplication.onActivityPaused(this);
    }

    @Override
    public void onDestroy()
    {
        super.onDestroy();
    }
}
{% endhighlight %}

Figuring out if your android application is in the foreground or background is logically not very difficult, but the SDK doesn't provide the easiest mechanisms to work with.  This example is one way to do it with relatively little modifications.

Sometime in the future I may look into handling things more immediately instead of deferring the check for foreground and background after half a second.
