---
layout: post
title: "Moving to the New Android Build System"
tags : [android, gradle, intellij, hg, git]
description : Switching from Eclipse and Ant to IntelliJ / Android Studio and Gradle
---
### New Android Build System
If you haven't seen / heard of the new android build system ( still work in progress ), then you should definitely go read up on it.  There is some great material on the [Android Tools Project Site](http://tools.android.com/tech-docs/new-build-system).

### Building Android projects with Ant
[Ant](http://ant.apache.com) has it's strengths, but the lack of IDE integration (can't run task and attach debugger and a pain to use with Eclipse) is really what has made me an early adopter of the new [Gradle](http://gradle.org) based build system.  I currently have an ant script that is powered by the default build that is packaged with the androd sdk.  I hijacked a few tasks to add the ```versionName``` and ```sourceVersion``` (build ID) to the output file.  Also, in the midst of the madness, I get the current revision number from [Mercurial](http://mercurial.selenic.com/) and use that as my build ID.  It has become very useful for traceability and for consistency in my builds.
**If you can't build your Android project with one command then you should look into it.**

### Ant / Eclipse -> Gradle / IntelliJ
This step was rather simple really.
1.	Open the Project in Eclipse
2.	File -> Export, Select Android -> Generate Gradle Build Files
3.	Open IntelliJ and Import Project
	+	Be sure to import from the Gradle files, not the Eclipse Project Files
	+	Use gradle wrapper, unless you have a compelling reason not to

At this point you have the basic gradle build files and should be able to run
{% highlight bash %}
./gradlew clean assembleDebug
{% endhighlight %}

#### Release Signing Application
In your ```build.gradle``` file, add a signingConfig using the keystore you have always been using for your apps.

{% highlight groovy %}
android{
    ...
    signingConfigs {
        release {
            storeFile file('MyReleaseKey.keystore')
            storePassword "123456"
            keyAlias "alias"
            keyPassword "password"
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
    ...
}
{% endhighlight %}

Now you can simply run
{% highlight bash %}
./gradlew clean assembleRelease
{% endhighlight %}

This will produce ```{Name of Project}-release.apk``` located in ```build/apk/```.

#### Adding Proguard
Adding proguard is very simple, but here is how to get it in there anyways.

{% highlight groovy %}
android{
    ...
    buildTypes {
        release {
            runProguard true
            proguardFile 'proguard.cfg'
        }
    }
    ...
}
{% endhighlight %}

If you run a release build you will now see all the proguard output.

### Integrating with SCM
Mercurial offers a few different commands that might be valuable in adding build numbers.  Most are a variant of ```hg id```, which can easily be executed through Gradle.  Using the revision number might not always be the best option.  Below I will show how to get the numeric revision number as well as the unique ID of the commit in Mercurial.  Also, I give an option for using the git hash.

<div class="alert alert-warning" style="font-size:1.25em;font-weight:normal;"><i class="icon-attention"> </i>Updated script to be much cleaner with advice from <a href="http://plus.google.com/u/1/+XavierDucrohet">Xavier Ducrohet</a></div>

You will notice below I write the build number into my ```versionName``` property.  Using advice from the [Android Developer Tools Community](https://plus.google.com/u/1/101836723454902363467/posts/3DszV82h6TN) I was able to change the output file for release builds to contain the ```versionName```.

{% highlight groovy %}
def getBuildNumber() {
	def os = new ByteArrayOutputStream()
	exec {
		// change to 'hg id -i' for the global revision id
		// change to 'git rev-parse HEAD' for git hash
		commandLine 'hg id -n'.split()
		standardOutput = os;
	}
	return os.toString().replaceAll("\\s", "")
}

android {
	defaultConfig {
		versionCode 13
		versionName "1.5.2 (" + getBuildNumber() + ")"
	}
	...
}

android.applicationVariants.each { variant ->
	if (variant.buildType.name.equals("release")) {
		variant.outputFile = file(
			"build/apk/"
			+ project.name + "_"
			+ android.defaultConfig.versionName + ".apk")
	}
}

{% endhighlight %}

<div style="text-decoration:line-through;margin-top:10px;">
You will notice below I write the build number in my strings.xml file.  Using advice from the [Android Developer Tools Community](https://plus.google.com/u/1/101836723454902363467/posts/3DszV82h6TN) I was able to change the output file of my release builds to contain both the ```versionName``` and ```sourceVersion``` (build ID).

{% highlight groovy %}

task getSourceVersion {
	def os = new ByteArrayOutputStream()
	exec{
		// change to 'hg id -i' for the global revision id
		// change to 'git rev-parse HEAD' for git hash
		commandLine 'hg id -n'.split()
		standardOutput = os;
	}
    project.ext.sourceVersion = os.toString().replaceAll("\\s","")
}

task updateVersionInfo(dependsOn: getSourceVersion) {
    println "updateVersionInfo"
    project.ext.strs = file('res/values/strings.xml');
    project.ext.strs.text = project.ext.strs.text.replaceFirst(
            'name="build">(.*?)<', 'name="build">' + project.ext.sourceVersion + '<');

    android.applicationVariants.each { variant ->
        if (variant.buildType.name.equals("release")) {
            variant.outputFile = file(
                    "build/apk/"
                    + project.name + "_"
                    + android.defaultConfig.versionName + "_"
                    + project.ext.sourceVersion + ".apk")
        }
    }
}

assembleRelease.dependsOn updateVersionInfo
installRelease.dependsOn updateVersionInfo

zipalignRelease.doLast {
    project.ext.strs.text = project.ext.strs.text.replaceFirst(
            'name="build">(.*?)<', 'name="build">-1<');
}

{% endhighlight %}



This is probably not the cleanest most efficient solution, but it works for the release builds I make.</div>


I can quickly get what I need just by executing.
{% highlight bash %}
./gradlew clean assembleRelease
{% endhighlight %}
