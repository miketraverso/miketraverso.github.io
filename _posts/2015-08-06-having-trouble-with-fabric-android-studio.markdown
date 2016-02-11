---
layout: post
title:  "Having trouble with Fabric and Android Studio?"
date:   2015-08-06 15:14:00
description: If you're having trouble getting the Fabric plugin to play nice with Android Studio this blog post will show you how to resolve those issues.
# categories: blog geolocation geofencing latitude longitude
tags: android UI UX libgdx programming developement gamedevelopment
thumbnail: /images/android-studio.png
---

Fabric, (www.fabric.io) is the new umbrella webapp from Twitter that incorporates all of Twitter's APIs. Fabric provides plugins for XCode, IntelliJ, Eclipse and Android Studio providing support for Crashlytics, Twitter, MoPub and Digits. It's extremely easy to get setup, most of the time that is, with it's walkthrough setup process. There are parts of the setup that either wait for you to do something and in most cases will actually update your files with the necessary code. No need to track down files, figure out where to make the change, make the change and hope for the best. 

Today though I came across a bit of a problem getting Fabric and the Crashlytics API properly installed in my version of Android Studio. A bit of background on my situation. I've got an Android only, libgdx based, project running with the latest stable Android Studio (v1.3.0.10). LibGDX allows you to create projects that run cross platform. Web, iOS, Android, Windows, Mac... heck even Blackberry. One of the nasty results of this is that there are multiple projects with multiple `build.gradle` files. Which probably led to my issue :

![Android Studio with Gradle build error]({{ site.baseurl }}/blog/images/gradle-error.png){: .img-responsive .center-block }

Basically my AndroidLauncher.java, which is my app's entry point, where I am calling the Fabric setup methods, complained that it couldn't find teh Crashlyitcs and Fabric packages. Ruh-roh right? 

{% highlight java %}
package com.traversoft.hff.android;

import android.os.Bundle;

import com.badlogic.gdx.backends.android.AndroidApplication;
import com.badlogic.gdx.backends.android.AndroidApplicationConfiguration;
import com.traversoft.hff.HereFishyFishyMain;
import com.crashlytics.android.Crashlytics;
import io.fabric.sdk.android.Fabric;

public class AndroidLauncher extends AndroidApplication {
	@Override
	protected void onCreate (Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		Fabric.with(this, new Crashlytics());
		AndroidApplicationConfiguration config = new AndroidApplicationConfiguration();
		initialize(new HereFishyFishyMain(), config);
	}
}
{% endhighlight%}

Something went wrong with the setup process. So let's check out our main build.gradle file.

{% highlight basemake %}

buildscript {
    repositories {
        mavenCentral()
        maven { url "https://oss.sonatype.org/content/repositories/snapshots/" }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'
    }
}

allprojects {
    apply plugin: "eclipse"
    apply plugin: "idea"

    version = '1.0'
    ext {
        appName = 'android-herefishyfishy'
        gdxVersion = '1.6.4'
        roboVMVersion = '1.5.0'
        box2DLightsVersion = '1.4'
        ashleyVersion = '1.4.0'
        aiVersion = '1.5.0'
    }

    repositories {
        mavenCentral()
        maven { url "https://oss.sonatype.org/content/repositories/snapshots/" }
        maven { url "https://oss.sonatype.org/content/repositories/releases/" }
    }
}

project(":android") {
    apply plugin: "android"

    configurations { natives }

    dependencies {
        compile project(":core")
        compile "com.badlogicgames.gdx:gdx-backend-android:$gdxVersion"
        natives "com.badlogicgames.gdx:gdx-platform:$gdxVersion:natives-armeabi"
        natives "com.badlogicgames.gdx:gdx-platform:$gdxVersion:natives-armeabi-v7a"
        natives "com.badlogicgames.gdx:gdx-platform:$gdxVersion:natives-x86"
        compile "com.badlogicgames.gdx:gdx-box2d:$gdxVersion"
        natives "com.badlogicgames.gdx:gdx-box2d-platform:$gdxVersion:natives-armeabi"
        natives "com.badlogicgames.gdx:gdx-box2d-platform:$gdxVersion:natives-armeabi-v7a"
        natives "com.badlogicgames.gdx:gdx-box2d-platform:$gdxVersion:natives-x86"
        compile "com.badlogicgames.gdx:gdx-freetype:$gdxVersion"
        natives "com.badlogicgames.gdx:gdx-freetype-platform:$gdxVersion:natives-armeabi"
        natives "com.badlogicgames.gdx:gdx-freetype-platform:$gdxVersion:natives-armeabi-v7a"
        natives "com.badlogicgames.gdx:gdx-freetype-platform:$gdxVersion:natives-x86"
        compile "com.badlogicgames.gdx:gdx-tools:$gdxVersion"
        compile fileTree(dir: '../libs', include: '*.jar')
    }
}

project(":core") {
    apply plugin: "java"


    dependencies {
        compile "com.badlogicgames.gdx:gdx:$gdxVersion"
        compile "com.badlogicgames.gdx:gdx-box2d:$gdxVersion"
        compile "net.dermetfan.libgdx-utils:libgdx-utils:0.13.0"
        compile "net.dermetfan.libgdx-utils:libgdx-utils-box2d:0.13.0"
        compile "com.kotcrab.vis:vis-ui:0.7.7"
        compile "com.underwaterapps.overlap2druntime:overlap2d-runtime-libgdx:0.1.0"
        compile "com.badlogicgames.gdx:gdx-freetype:$gdxVersion"
        compile "com.badlogicgames.gdx:gdx-tools:$gdxVersion"
        compile fileTree(dir: '../libs', include: '*.jar')
    }
}

tasks.eclipse.doLast {
    delete ".project"
}
{% endhighlight %}

So the problem here is that there is no mention anywhere to getting the necessary jars for a complete Fabric/Crashlytics installation. So let's go add it. First off we will need to add repositories and dependencies to the buildscript section of the build.gradle file

{%highlight text%}
buildscript {
    repositories {
        mavenCentral()
        jcenter()
        maven { url "https://oss.sonatype.org/content/repositories/snapshots/" }
        maven { url 'https://maven.fabric.io/public' }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'
        classpath 'io.fabric.tools:gradle:1.+'
    }
}
{%endhighlight%}

`jcenter()` refers to the new hotness that is a new maven repository for Android Studio's gradle plugin. [Check it out here](https://bintray.com/bintray/jcenter). Then we have the Fabric specific maven url. Under dependencies we see the Fabric classpath pointing to the latest version 1.+. Next we look to apply our Fabric plugin. In my case it's under `project(":android")` if you're just using a basic android app gradle.build this probably won't be within a `project` block and would just normally be under the `buildscript` section.     

{%highlight text%}
project(":android") {
    apply plugin: "android"
    apply plugin: 'io.fabric'

    configurations { natives }

    repositories {
        jcenter()
        maven { url 'https://maven.fabric.io/public' }
    }

	...
{%endhighlight%}

Here we re adding our `apply plugin: 'io.fabric'` command and adding our repositories block. So that covers what Twitter covers in their manual setup [blog](http://docs.fabric.io/android/fabric/integration.html). If you sync up your gradle build now you should be all good to go... except you won't be. The Twitter blog fails to mention that you also need to add the following under your Android app's `dependencies` section.

{%highlight text%}
compile('com.crashlytics.sdk.android:crashlytics:2.2.3@aar') {
    transitive = true;
} 
{%endhighlight%}

We're specifying `transitive = true;` to get any dependencies that the crashlytics aar depends upon. (FYI: An AAR is the binary distribution of an Android Library Project, like an **A**ndroid **AR**chive). Once this is added we're all done and we'll notice if we gradle sync now our entry point class, or wherever you've added `Fabric.with(this, new Crashlytics());` will now be able to find the necessary libraries it needs to compile. Here's a full listing of my build.gradle for reference. 

Hope that this helps!

{%highlight make%}
buildscript {
    repositories {
        mavenCentral()
        jcenter()
        maven { url "https://oss.sonatype.org/content/repositories/snapshots/" }
        maven { url 'https://maven.fabric.io/public' }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'
        classpath 'io.fabric.tools:gradle:1.+'
    }
}

allprojects {
    apply plugin: "eclipse"
    apply plugin: "idea"

    version = '1.0'
    ext {
        appName = 'android-herefishyfishy'
        gdxVersion = '1.6.4'
        roboVMVersion = '1.5.0'
        box2DLightsVersion = '1.4'
        ashleyVersion = '1.4.0'
        aiVersion = '1.5.0'
    }

    repositories {
        mavenCentral()
        maven { url "https://oss.sonatype.org/content/repositories/snapshots/" }
        maven { url "https://oss.sonatype.org/content/repositories/releases/" }
    }
}

project(":android") {
    apply plugin: "android"
    apply plugin: 'io.fabric'

    configurations { natives }

    repositories {
        jcenter()
        maven { url 'https://maven.fabric.io/public' }
    }

    dependencies {
        compile project(":core")
        compile "com.badlogicgames.gdx:gdx-backend-android:$gdxVersion"
        natives "com.badlogicgames.gdx:gdx-platform:$gdxVersion:natives-armeabi"
        natives "com.badlogicgames.gdx:gdx-platform:$gdxVersion:natives-armeabi-v7a"
        natives "com.badlogicgames.gdx:gdx-platform:$gdxVersion:natives-x86"
        compile "com.badlogicgames.gdx:gdx-box2d:$gdxVersion"
        natives "com.badlogicgames.gdx:gdx-box2d-platform:$gdxVersion:natives-armeabi"
        natives "com.badlogicgames.gdx:gdx-box2d-platform:$gdxVersion:natives-armeabi-v7a"
        natives "com.badlogicgames.gdx:gdx-box2d-platform:$gdxVersion:natives-x86"
        compile "com.badlogicgames.gdx:gdx-freetype:$gdxVersion"
        natives "com.badlogicgames.gdx:gdx-freetype-platform:$gdxVersion:natives-armeabi"
        natives "com.badlogicgames.gdx:gdx-freetype-platform:$gdxVersion:natives-armeabi-v7a"
        natives "com.badlogicgames.gdx:gdx-freetype-platform:$gdxVersion:natives-x86"
        compile "com.badlogicgames.gdx:gdx-tools:$gdxVersion"
        compile fileTree(dir: '../libs', include: '*.jar')
        compile('com.crashlytics.sdk.android:crashlytics:2.2.3@aar') {
            transitive = true;
        }
    }
}

project(":core") {
    apply plugin: "java"


    dependencies {
        compile "com.badlogicgames.gdx:gdx:$gdxVersion"
        compile "com.badlogicgames.gdx:gdx-box2d:$gdxVersion"
        compile "net.dermetfan.libgdx-utils:libgdx-utils:0.13.0"
        compile "net.dermetfan.libgdx-utils:libgdx-utils-box2d:0.13.0"
        compile "com.kotcrab.vis:vis-ui:0.7.7"
        compile "com.underwaterapps.overlap2druntime:overlap2d-runtime-libgdx:0.1.0"
        compile "com.badlogicgames.gdx:gdx-freetype:$gdxVersion"
        compile "com.badlogicgames.gdx:gdx-tools:$gdxVersion"
        compile fileTree(dir: '../libs', include: '*.jar')
    }
}

tasks.eclipse.doLast {
    delete ".project"
}
{%endhighlight%}
