---
layout: post
title: "Getting Networking Out of the Activity Lifecycle"
date: 2014-09-29
categories: android fragment networking rxjava otto bus rxjava
---
When building a REST Client Android app, we often encounter issues as a result of trying to perform asynchronous operations from inside an Activity. Since we can't perform network requests on the UI thread, we must offload them to background threads, then repopulate the UI on the main thread with the result of our requests. This opens up the floodgates for memory leaks, duplicate requests, and ugly code. There will *always* be issues related to the Activity life cycle -- especially if we're doing all our requests inside of an Activity.

For me, the three obvious substitutes for networking inside of an Activity are **Retained Fragments**, **Services**, and the **Application Context**.

The retained Fragment approach [is described here by Alex Lockwood](http://www.androiddesignpatterns.com/2013/04/retaining-objects-across-config-changes.html). We create a Fragment inside of an Activity and call setRetainInstance(true) inside of its onCreate() method. As a result, the Fragment is kept alive during configuration changes in the host Activity. We get no help on threads here, but this can be fixed with your asynchronous networking library of choice.

To be honest, retained Fragments are the approach that I've explored the least. I tend to shy away from Fragments in general (for UI purposes or otherwise) as a result of their complexity.

Services allow us to do work (i.e. a network request) that can be initiated from an Activity but not bound to the Activity life cycle. If we use an IntentService, which inherits from Service, we don't even have to be worry about threads -- IntentServices run in a background thread by default.

However, Services are clunky and require quite a bit of boilerplate just to make one simple request. We can clean up the actual request code quite a bit with a networking library like Retrofit, but there's still the unbundling of the Intent data to deal with. There's also the issue of communicating data back to the Activity to populate the UI.  I'm partial to Otto (an event bus by Square) for this use case.

We also have the option of performing our work in the Application Context, as described in [this blog post](http://www.mdswanson.com/blog/2014/04/07/durable-android-rest-clients.html) by Matt Swanson. We create a POJO in our Application subclass that subscribes to events posted via an event bus. When an event is fired, we start an asynchronous network request using Retrofit with a Callback.  In the onSuccess() or onFailure() methods in the request Callback, we post an event to whom an Activity or UI component may subscribe.

In my opinion, this last approach is quite a bit cleaner than using an IntentService -- less boilerplate and no pesky Intents with several putExtra() calls. It relies heavily on Otto, but I think that's a good thing. There's something very appealing about having the networking code completely decoupled from the UI code.

All three of the above approaches move all background work away from the Activity life cycle, which helps us avoid Context leaks across configuration changes and write cleaner, more modular code. At the moment, my favorite is the Application Context approach, but I feel there's still room for improvement there -- perhaps with an asynchronous library like RxJava.