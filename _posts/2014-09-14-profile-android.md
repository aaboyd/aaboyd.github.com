---
layout: post
title: "Profile Your Android App Before You Need To"
tags : [java, android]
description : Learn the android tools and use them before you need to.
---

## Android Tools

The android sdk comes with a few tools that aren't used as often as the main IDE [Android Studio](http://developer.android.com/sdk/installing/studio.html) or [Eclipse ADT](http://developer.android.com/tools/sdk/eclipse-adt.html).  Some of these tools are not included with Android Studio and must be run individually.

<div class="alert success">The rest of this post will assume users are using Android Studio.  The methodology / workflow is a bit different using Eclipse ADT</div>

It's important to learn and use these tools before you **_NEED_** them.

The two tools I have used in the past and had great success with are :

1. [Heap Dump Analyzer](#heapanalyzer)
2. [Thread & Method Profiler](#methodprofiler)

## Heap Analayzer

### How this can help

The heap analyzer can be used to view what memory is allocated.  It should help you identify any memory leaks.  Yes, Java has a garbage collector, but No, that does not make it immune to memory leaks.  Even if you aren't too concerned with memory leaks, it's great to get a look at what is actually going on under the hood.

### Why I started using it

A few years ago, I inherited an android application that was occasionally spitting out some unexpected errors to Logcat.  In my particular case there were some memory leaks that were fairly simple to fix.  The app was (it's not as bad anymore) filled with ```static``` variables.  Those variables led to leaking android activities.  Using static variables aren't the greatest habit to develop, and it's a pain to try and fix after the fact.

### Using the tools

#### Open up the tools
<img alt="Android Studio Device Monitor" style="width:212px;height:52px" src="/assets/img/profile-android/android_monitor.png" />

Located at the top toolbar of Android Studio you will find a button to open the [Android Device Monitor](http://developer.android.com/tools/help/monitor.html).

#### Dump hprof to file
![Dump Hprof](/assets/img/profile-android/dump_hprof.png)

In the "Devices" pane on the left, it should show the devices that are plugged in and visible to [adb](http://developer.android.com/tools/help/adb.html).  In the tree there is a list of applications that are debuggable.  Select the one you want to get the heap from and select the "dump hprof" button (green cylinder with a red down arrow).

#### Convert hprof

The hprof file you saved needs to be converted using the [Android Hprof Converter](http://developer.android.com/tools/help/hprof-conv.html).  This tool can be found in the platform-tools directory of you android sdk.

The tool simply converts the hprof to be imported to other tools.  In my case I used [MAT](http://www.eclipse.org/mat/).

``` shell
hprof-conv input.hprof output.hprof
```

#### View Results in MAT

##### Download Standalone MAT
[Download](http://www.eclipse.org/mat/downloads.php) the appropriate standalone MAT for your operating system.

##### Import the hprof

1. Open up the standalone MemoryAnalyzer.
2. Select File->Open Heap Dump...
3. Select the output file from hprof-conv
4. Select options in the heap dump "Getting Started Wizard"
<img alt="MAT Getting Started Wizard" style="height:355px;width:567px" src="/assets/img/profile-android/mat_getting_started.png" />
5. Analyze results

For more info on how to analyze the results and different things to look for.  Check out the [MAT Documentation](http://help.eclipse.org/luna/index.jsp?topic=/org.eclipse.mat.ui.help/welcome.html).

## Method Profiler

### How this can help
The method profiler can show you exactly what methods are taking the longest CPU time.  It can also show you when threads are created and what work is being done on each.  A small method can turn into a large bottleneck, if it's never optimized.

### Why I started using it
I noticed some very small periods of time in the application that the UI was unresponsive.  The portion of code that was causing the problems wasn't hard to find.  Instead of just offloading the work to a background thread ( which I did end up doing ), I found a few hidden methods that helped improve speed greatly.

#### Open up the tools
<img alt="Android Studio Device Monitor" style="width:212px;height:52px" src="/assets/img/profile-android/android_monitor.png" />

Located at the top toolbar of Android Studio you will find a button to open the [Android Device Monitor](http://developer.android.com/tools/help/monitor.html).

#### Start Method Profiling
<img alt="Start Method Profiling" style="width:386px;height:146px;" src="/assets/img/profile-android/start_method_profiling.png" />

In the "Devices" pane on the left, it should show the devices that are plugged in and visible to [adb](http://developer.android.com/tools/help/adb.html).  In the tree there is a list of applications that are debuggable.  Select the one you want to get method information from and select the "start method profiling" button (three arrows with a red dot in the top right corner).

After starting method profiling.  Use the app a bit to get some data.  If you have a particular problem area.  You will want to start profiling right before you hit that code.

Immediately following the use of the app.  Go ahead and hit the "stop method profiling" button.  It's the same button as you hit to start profiling.

#### View Results
The results should automatically be showed inside device monitor.  The main view shows which methods are running the longest and a graph at the top showing what threads are used and what methods are run on each thread.

For more info on how to analyze the results and different things to look for.  Check out the [Android Profiling Documentation](http://developer.android.com/tools/debugging/debugging-tracing.html).


<div class="alert secondary">Expect a follow up with details from an improvement I made in less than 20 minutes becuase of these tools</div>
