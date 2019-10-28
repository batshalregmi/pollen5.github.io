---
title: "Building Android apps by hand."
layout: post
category: android
tags: android java diy
published: false
---

Have you built an Android app? i bet you used an IDE like Android Studio and a build system like Gradle to do it, in this post we will build an app manually by hand and see what's actually happening behind the scenes.

### Get off your IDE
First step to follow this guide is to get off your damn IDE and open a basic text editor, Vim/Nano works fine, heck feel free to even use Notepad.

### Open your Terminal
Open a terminal, whether it's your cmd.exe or your beautiful customized hacker shell on Linux, we'll need to run commands to do this.

### Android SDK
for android sdk we only need a few things, an android.jar that holds all Android's java stdlib for the API level you are targetting and the command line tools, we will need `dx`/`aapt` mainly.

We will also need a java compiler but if you used an IDE you probably already installed the JDK and stuff.

Lastly we will need an apk signer, `jarsigner` that comes with java is fine.

### File structure
This is the structure i use:
```
.
├── AndroidManifest.xml
├── libs
│   └── somelib.jar
├── res
│   └── drawable
└── src
    └── main
        └── java
            └── com
                └── pollen
                    └── blog
                        └── MainActivity.java

9 directories, 3 files
```
- `src/main/java` holds all the Java sources `com/pollen/blog` path is the package for a java `package com.pollen.blog;`
- `res` holds resources that are packaged in the APK.
- `res/drawable` holds the image for your App's icon.
- `res/layout` Holds XML layouts for the UI.
- `AndroidManifest.xml` have the manifest for your app, it won't be covered here.
- `libs` Holds external jar file dependencies to use in our app.
- `libs/somelib.jar` a dummy jar just for the sake of this tutorial.

Things that you should already know but just giving a summary.

### MainActivity.java
{% highlight java %}
package com.pollen.blog;

import android.app.Activity;
import android.os.Bundle;

public class MainActivity extends Activity {

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main);
  }
}
{% endhighlight %}
Nothing too fancy here, just a basic Android app that loads an xml layout.

### AndroidManifest.xml
For this to be a truly do-it-yourself i have to show you this file too rather than let you generate it with your IDE.
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.pollen.blog">
   <application
      android:allowBackup="true"
      android:label="Blog Tutorial"
      android:icon="@drawable/icon.jpg"
      android:debuggable="true"
      android:supportsRtl="true">
      <activity android:name=".MainActivity">
         <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
         </intent-filter>
       </activity>
   </application>
 </manifest>
{% endhighlight %}
I won't go over all what it does but take it with a grain of salt, you may also need to remove the `android:icon` line if you did not add an `icon.jpg` to `res/drawable`

### libs/somelib.jar
You may wonder what this is, what do i use to follow the tutorial, answer is it's nothing, i created an empty file just to show you how adding external jars work, if you have a library to use put it in here (e.g `okhttp.jar`)

### The Pipeline
In order to do this we have to first understand how the pipeline works, it goes like:
```
.java source -> [java compiler, e.g javac] -> .class files -> [dx] -> .dex file -> [DalvikVM]
```
Stages wrapped in `[]` are ones that *consumes* the output by the previous stage.

That's the general idea of how building Java files work plus how android does it.

Android uses the special `.dex` files which stores instructions for the Dalvik Virtual Machine, so we have to convert the output class files to Android's dex file, dex also packs all of them into a single dex file, called `classes.dex` in the root of an APK.

That's building the java sources but what about packaging the APK?
```
aapt -> generates R.java -> [javac] -> .class -> [dx] -> Initial apk file that holds the classes.dex -> [aapt] -> Adds resources to the apk -> [jarsigner] -> Signs the apk.
```
It's worth mentioning that other tutorials creates the initial apk with aapt too but we do it with dx because dx will also package resources from external jars while aapt doesn't, a big headache i went through when i was following others.


