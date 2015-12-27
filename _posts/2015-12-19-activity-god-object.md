---
layout: post
title:  The Activity god object at the heart of Android
date:   2015-12-19 14:05:16
categories: android mvp presenter architecture activity lifecycle
---
When I first started developing Android apps, I have to say I was baffled and genuinely confused by the practices regarding application structure and architecture that were being taught in introductory tutorials, books and other resources. I think my confusion was best summed up by a quote from [this great blog post from Square][square]:
[square]: https://corner.squareup.com/2014/10/advocating-against-android-fragments.html

>On Android, Context is a god object, and Activity is a context with extra lifecycle. A god with lifecycle? Kind of ironic. Fragments aren’t gods, but they make up for it by having extremely complex lifecycle.

I would further add to this by saying that not only are Activities/Fragments treated as god objects by the Android APIs, but they are also then extended and treated as a god object for the app logic by the developers building apps on top of them.

Activities and Fragments can often be filled with view logic, business logic, network logic and database logic all in one place, woven across multiple threads and destroyed at any moment by a screen rotation. Even if the code is slightly cleaner and split across multiple classes, those classes often have a tight coupling to the Activity/Fragment god object and their lifetime is tied to the lifetime of the god object.

The practice of treating an Activity/Fragment as a god object for a large portion of the app is not just a simplification for beginners to get started. From what I’ve seen during my admittedly short time in the industry, it’s a very common practice. Each Activity/Fragment is almost an entire separate application within your application that lives and dies in a way that is unpredictable and sudden (such as a screen orientation change). Apps are structured in terms of how the UI has been partitioned rather than in a way that suites the business logic.

##Memory leaks
Developers are being asked to build more complex apps. Using the god object anti-pattern practice to build these apps is insanity. It will probably lead to spaghetti code that’s very difficult to follow, and inside that spaghetti the reference chains will inevitably grow as the code base does.

It can lead to problems such as callbacks across threads (such as in a `Runnable` or `AsyncTask`) leaving a reference chain back to a dead activity, which not only temporarily leaks the Activity, but if the Activity is assumed to still be alive on callback termination, could crash the whole app.

Even worse, it can encourage the use of static state to escape the Activity/Fragment lifecycle, and that static state can easily cause memory leaks that last the lifetime of the app. The creation of singletons using a public static variable has long been considered an anti-pattern in Java, and it’s no surprise that it found it’s way into Android apps as an attempt to escape the chaos of the Activity lifecycle. Have you ever cached an object in static state that required a context to construct? If you used the Activity `Context` to construct the object then you're leaking the Activity. 

##Threading concerns
Beginners in Android development are not taught about how to architect a larger application that exists outside of the god objects and their lifecycles. However, they are quickly taught to move networking off of the main thread, shortly followed by database calls and other long running tasks. This keeps the main thread free to serve a responsive UI experience. However, the combination of god objects + multithreading + the rapidly increasing complexity of mobile apps can be total architectural chaos.

Without a very clearly defined architecture regarding threading, you could end up in situations where you have no idea which objects currently exist, which thread you are currently on, or what the lifecycle of the object you are inside of is. A horrible anti-pattern I’ve seen for making sure some code isn’t running on the UI thread is to just often code “defensively” with a `Runnable` inside a thread pool for any logic that is suspected to be long running at some level in that stack, and then eventually get results back to the UI by placing another `Runnable` onto the main thread. Chaos.

##Spaghetti code: endorsed by Google?
Google seem to care very little about actually helping developers write well structured modern Android apps that are larger than hobby sized projects. Google’s resources for learning Android all teach a spaghetti architecture where everything hangs off the Activity/Fragment god object.

This wasn’t just to make it easy for beginners; they offer no real resources even for experts to learn a more scalable app architecture. I think that part of the problem is that there is still a widely held belief that mobile apps *are* still small weekend projects which of course hasn't been true for a long time.

The only large example app Google publish the source code for is [the Google I/O app][ioapp] which also uses the same spaghetti architecture. The 2015 iteration is admittedly a little better architecturally than previous iterations and has some version of an MVP architecture but I haven’t spent much time going through the source since confirming that it is essentially still a mess. There are better github projects to spend your time learning from (I’ll mention them later).
[ioapp]: https://github.com/google/iosched

One minor point: Google is clearly on-board with regard to dependency injection on Android. They forked [Square’s Dagger dependency injector][dagger1] to create [Dagger 2][dagger2], creating much confusion in the process over the relative advantages and disadvantages of each which I won’t go into. So why wasn’t it present in this year’s I/O app? Who knows. Dependency Injection is part of the solution to this whole mess and I really like Dagger 2 in particular because it allows you to define different injection scopes (a topic for another day).
[dagger1]: http://square.github.io/dagger/
[dagger2]: http://google.github.io/dagger/

## "Why do Android apps crash so much?"
Considering Google’s investment in the platform, I find it puzzling that they are content with god objects and spaghetti code as the norm on Android. The Android team made it so that an exception is thrown if you attempt to do networking on the main thread, which forces the developer to keep the UI responsive. Keeping Android apps free of crashes and memory leaks is just as important and will never happen without better architectural practices and assistance available to developers both in the form of educational resources and better libraries. 

The problem of poor architecture built around god objects is endemic in Android app development because first time developers are taught it as a best practice. A few years ago I worked on a large Google Web Toolkit app and strangely back then Google knew this. All of their example apps for Google Web Toolkit used an MVP architecture and the Gin dependency injector to keep everything well separated. There was no god object that I recall. The large web app that we built just used the same architecture as the example projects, and it scaled up just fine. 

I suspect that Apple is more aware of this on iOS. A friend of mine recently got his first Android device and asked me why apps crash so much more on Android than on iOS. I don’t use iOS or develop iOS apps so I don’t have a good answer. But from conversations with my iOS developer coworker at work, it seems like Apple provides core libraries for taking care of most of the stuff I’ve had to either code myself of find third party libraries on github for.

##The solution: Dependency Injection and Model-View-Presenter
Thankfully, everything required to produce well structured Android apps is already available thanks to the heroic efforts of third parties to fill the gaping holes in the Android SDK. But it does seem strange that the Android SDK can’t even provide tools as basic as an event bus or dependency injector.

As for educational resources, currently the only way to learn decent scalable Android architecture is to find the right projects on github and go through the source (a great way to learn either way). 
Here are some of my favourite example sample architecture projects:

* [Android-CleanArchitecture][cleanarch] - My favourite one. If you aren’t looking to use rxJava then try going back through earlier commits to find a point before it was added to the project. There is also an interesting point before he even adds a dependency injector, just to demonstrate what a mess it would be without one.
[cleanarch]:https://github.com/android10/Android-CleanArchitecture

* [MvpCleanArchitecture][mvp] - I also found this one useful, it has an example of paging across a large dataset while remaining decoupled from the database for instance (not perfect but it’s a good starting point).
[mvp]:https://github.com/glomadrian/MvpCleanArchitecture

* [EffectiveAndroidUI][effective] - Also a good example.
[effective]:https://github.com/pedrovgs/EffectiveAndroidUI

And my favourite libraries for helping with implementing these architectures:

* [Dagger 2][dagger2] - My preferred dependency injector for Android. It uses generated code so it’s super fast and gives injection errors at compile time making it easier to debug. It also allows you to have several different injection scopes. The [Clean Architecture project above][cleanarch] uses it, and is a good example for how to get it set up.

* [android-priority-jobqueue][job] - While I use the [Clean Architecture][cleanarch] shown above, my domain layer Use Cases are `Job`’s from this library rather than plain `Runnable`’s. They have advantages such as the ability to set the priority of jobs and queue jobs that require network access.
[job]:https://github.com/yigit/android-priority-jobqueue

*  [EventBus][eventbus] - My preferred event bus over [Otto][otto] because it allows you to easily subscribe to events posted on different threads. I use this for communication rather than callbacks since it’s more flexible and makes cross-thread communication so easy. Plus, you can easily unsubscribe from it and avoid the potential memory leaks that callbacks open up.
[eventbus]:https://github.com/greenrobot/EventBus
[otto]:http://square.github.io/otto/

*  Special mention: [RxJava][rx] this could replace the event bus part of the architecture, and reactive functional programming is a fairly fundamental change that opens up a whole new set of architectural possibilities if it is fully embraced. It can also be used to loosely couple your View object to your Presenter object for example. Simply put, I just don’t have enough experience to use it in production yet. I definitely plan to move towards it in the future though.
[rx]:https://github.com/ReactiveX/RxJava

I will also hopefully find time to write a post about my preferred MVP architecture and proposed solution to the Activity lifecycle problem, which these projects don’t really cover.
