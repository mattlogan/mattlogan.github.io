---
layout: post
title: "Replacing Otto with RxJava"
date: 2015-7-16
categories: android mvp architecture otto rxjava
---
*programmers paradox: repeatedly taking the simplest option leads to greatest complexity* - @tabdulla

I've recently replaced Otto with RxJava in the data layer of my current Android project. Otto allows us to perform asynchronous data requests outside of the Activity lifecycle with a simple publish/subscribe event-based API. However, replacing Otto with RxJava has removed tedious bookkeeping and boilerplate while paving the way for code that is more direct and less error-prone.

**Why do we need any of this?**

Updating an app's UI with the result of an asynchronous data request is risky business. If we call a data request method with a callback argument from inside an Activity -- or anything that lives inside an Activity, including Fragments, Views, or some other delegate object -- and the callback populates the UI with data, that callback retains a reference to the Activity.

If the Activity is destroyed before the result is delivered to the callback, the callback outlives the Activity and the Activity is leaked. Additionally, if we try to call methods on Views after they've been garbage collected, we can get NullPointerExceptions that crash our app. The naive fix is to add null checks before updating the view, but this is less than ideal!

This is the first thing we point to when we talk about why "AsyncTask is bad," but really it's an issue with any callback that's retained beyond Activity destruction. Ideally, we'd like to completely ignore, or "unsubscribe" from, the result of a data request so we can avoid Context leaks and ugly null checks. (In the *super* ideal case, we can also use the result of the initial request to populate the UI after the Activity is recreated, but I won't get into this topic in this post.)

**The Otto approach**

My first foray into getting asynchronous data requests outside of the Activity lifecycle started with a [blog post](http://www.mdswanson.com/blog/2014/04/07/durable-android-rest-clients.html) that describes a way to achieve this using Otto. Otto gives us a "bus" singleton that can post "events" (i.e. instances of an event class, like LoadStoriesEvent). Any object can "subscribe" to and "unsubscribe" from specific events posted by the bus.

The Otto implementation in an Android app works like this: an Activity uses the bus to post an event requesting a specific kind of data. Another object that lives outside of the Activity lifecycle (for example in the Application object) is subscribed to this event. This object then calls an asynchronous data request method with a callback argument. In a callback method, it uses the bus to post the result data. The Activity subscribes to the result event (success or failure), and updates the UI with the result. The Activity subscribes to events in onResume() and unsubscribes in onPause(). 

If the request outlives the Activity, the Activity is not leaked because the bus releases its Activity reference when the Activity unsubscribes itself in onPause(). Additionally, the callback has no reference to the Activity -- just the bus. A while back I wrote an [example app demonstrating this implementation in practice](https://github.com/mattlogan/RhymeCity).

While this approach is dead simple in theory, it can blow up quickly in complexity as an app grows. First, let's say we have an app that can request data from ten different endpoints. We already need thirty event classes! Each data request needs a request event, a success event, and a failure event. Second, it's non-trivial to perform a data request and use the result of that request to perform a second, or even third, request. The code path from user input to displaying the result becomes increasingly difficult to trace (and test!) as the number of dependent requests increases. This is further complicated if we need to manipulate or filter result data before posting the next request event. Third, we need a lot of code to keep track of all this -- subscribing, unsubscribing, posting events, subscribing to success and failure events, caching intermediary results, and filtering data between requests or before populating the UI.

**The RxJava fix**

RxJava addresses all of these issues by allowing us to create a composable "stream" of data that can be filtered, combined, cached, or manipulated in pretty much any way we want -- all in one expression, or "observable sequence." We don't need thirty event classes anymore. Combining multiple dependent requests is literally built-in. And because we can do all our data processing (and did I mention error handling?) in a single stream, there's very little bookkeeping that needs to be done.

Don't get me wrong -- RxJava is a complicated beast. Looking at its source code gives me a headache, whereas Otto's source is actually pretty simple. RxJava isn't the simplest option for getting asynchronous data requests out of the Activity lifecycle, but in my opinion it obviates complexity through its powerful streaming API.