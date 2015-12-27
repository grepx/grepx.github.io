---
layout: post
title:  Application Context != Activity Context
date:   2015-12-27 14:05:16
categories: android activity application context
---
This is something that took me a while to wrap my head around when I first started developing for Android.

## What is a `Context`?
The first Context object you will run into when you start developing Android apps will be the Activity Context. It’s a very strange object. It kind of represents the connection back to the the device running your app, as if your app is running remotely in some far off place away from the device. 

Everything seems to depend on Context and to confuse you further, the Context object, *is* the Activity object that you’ve subclassed and are filling with your own code.

Need to inflate a View layout xml file? Use the Context. Construct a button? Use the Context. Access a SQLite database? Use the Context. Access a String resource? Use the Context.

## Application Context
Just as you are getting your head around that, you learn that there is another Context object floating around in your app, the Application Context. You quickly learn that the Application Context is useful because you can stick it in a global static singleton like this:

```java
public class MainApplication extends Application {

  public static Context APP_CONTEXT;

  @Override
  public void onCreate() {
    super.onCreate();
    APP_CONTEXT = this;
  }
}
```

Then when you are deep down in some class, far away from the current Activity and need to access a String resource, for instance, you can do this:

```java
MainApplication.APP_CONTEXT.getResources().getString(R.string.hello_world);
```

And not have to worry about trying to find a way to get the Activity Context all the way down to that class. But do you know *why* you are "allowed" to do that? Do you know what this new Application Context object is?

For instance, why are you **not** allowed to do this, and get the Activity Context?

```java
public class MainActivity extends AppCompatActivity {

  public static Context ACTIVITY_CONTEXT;
  
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    ACTIVITY_CONTEXT = this;
  }
}
```

The answer lies in the fact that the Activity Context **is** your Activity, as mentioned earlier.

Anything placed inside a static variable lives forever (or until the reference is overwritten) in Java/Android. So if you were to put the Activity Context into a static variable, there is a good chance you would cause an Activity memory leak unless you manually cleaned it up by setting the reference to `null` later, which could then cause a crash if another part of the app assumed it wasn’t `null`.

##...so why are there 2 different Context objects?

The original design vision for Android seems to have been around the idea that an Activity is really a mini app inside an app. This was an interesting idea and some really nice stuff comes out of it, such as launching an Activity from another app to handle a use case that your app doesn’t handle via an [Implicit Intent][intent]
[intent]:http://codetheory.in/android-intents/

However, Activities are also *not really* entire apps, so the `Application` class can also be subclassed to give a an object that is more traditionally what a developer would expect in terms of lifetime and scope. Always remember: Activities are totally destroyed on a device screen rotation, we need something a bit more stable to work with.

So we have these 2 objects, the Application (the app) and the Activity (an app inside the app) that have some similarities, but some differences - and the similarities ended up being expressed by the fact that both are `Context` subclasses.

The reason you are “allowed” to put the Application Context object into a static singleton and grab it from all over the app is that it will live for the entire life of the application anyway, unlike Activities which are constantly being disposed of (for example, on a screen rotation). However, it should be used very sparingly, if at all. Mainly because it destroys the unit testability of any class that uses it, which now depends on having a fully initialised Application Context, which requires the entire Android stack to be running.

## Application Context != Activity Context (requiem)
You need to be aware that the Application Context != Activity Context. You cannot build UI with the Application Context for instance.

Generally speaking you should be using the Activity Context for most stuff that needs a Context. If you are creating an object that should last the lifetime of the application however, such as a SQLite database connection, then you should always use the Application Context, or you will memory leak the Activity that created the object.



 