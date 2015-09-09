---
layout: post
title: "Decoupling the Presenter"
date: 2014-10-27
---
The model-view-presenter design pattern for Android apps, or MVP, allows us to write testable POJOs that we call presenters containing code that isn’t tied to the view. In this case, the "view" can be an Activity, a Fragment, or a View – anything responsible for setting the text on a TextView or setting a loading indicator’s visibility, for example. We can inject dependencies like an event bus, API client, or database manager into the presenter via its constructor or an alternative dependency injection technique like Dagger or Guice. These dependencies can be mocked in a test class with Mockito, and we can unit test the presenter until the cows come home.

The need for presenters is further exacerbated by the fact that Activities and Fragments must have empty constructors. Dependency injection via constructor is therefore off the table. If you want to inflate a View via XML, you can’t override its constructor – resulting in the exact same problem.

In practice, we instantiate a presenter object in an Activity or Fragment’s onCreate() method or a View’s constructor or onFinishInflate(). Dependencies are injected and we’re off to the races. A View’s onClick() is triggered and we call a method in the presenter to do some stuff. Some stuff is done in the presenter, and we show the result in the view.

But how exactly do we show anything in the view from the presenter? Does the presenter then need a reference to the view? If a presenter has a reference to the view that instantiated it, it’s not really separate at all -- it's still directly coupled to the view.

The best solution, in my opinion, is to hide the view behind an Interface, which I like to call a ViewContract. If the view part of the MVP -- an Activity, Fragment, or View -- implements this ViewContract, and the presenter only requires an instance of the ViewContract, the presenter is effectively decoupled from the view.

The view could be anything -- Activity, Fragment, View, or even an anonymous inner class inside of one of these classes -- as long as it implements the ViewContract Interface. The presenter doesn't need to know the inner workings of a big heavy view object -- just a thin set of methods that it knows it can call to display its results in the view.

Furthermore, in a test class we wouldn't even need a sophisticated mocking library like Mockito to satisfy dependencies. We could simply instantiate our ViewContract interface with empty method implementations and inject this into the presenter.

Even if testability isn't your primary concern, this is still good design. Creating an object with a reference to the object that instantiated it is not a good practice -- this is why Java has Interfaces.