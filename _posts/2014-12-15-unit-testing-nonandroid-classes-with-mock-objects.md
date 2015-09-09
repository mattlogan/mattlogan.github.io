---
layout: post
title: "Unit Testing Non-Android Classes with Mock Objects"
date: 2014-12-15
---
The Android Framework provides a slew of classes for us to use in a multitude of ways: inheritance (Activity, Fragment, or Service), delegation (Handler, Looper, or LayoutInflater), or some combination (View, etc.). But for a good chunk of our app logic, we don't need any of this stuff. This is the logic we can best unit test, and mock objects go a long way to help with this.

We can create plain Java classes (classes that don't inherit from an Android Framework class) to perform tasks like posting events to an [event bus](http://square.github.io/otto/), iterating through stored object collections, and populating the UI with data through a View interface -- [think Model-View-Presenter](http://mattlogan.me/decoupling-the-presenter)! In this example, the event bus, data storage interface, and view interface (all plain Java classes and interfaces) can be injected*, and can therefore be mocked!

In a unit test class, we create an instance of our Java class under test. To satisfy its dependencies, we can instantiate mocks with Mockito and supply them as constructor arguments. We can leverage the insane power of Mockito to verify the behavior of each of the public methods in our class under test.

For example, we could call some method named onLoadDataRequested() in a Presenter and use Mockito to make sure a) data is loaded from our data storage via its injected interface, b) a call to our injected view interface is made to populate the UI, and c) the arguments of this UI method call are correct. (I'm sticking with the MVP UI pattern for this example, but this could apply to any Java class.)

My Mockit toolbox most commonly includes:

1. `Mockito.when()` -- We can use when() with thenReturn() to create predetermined responses to method calls on our mock objects.

2. `Mockito.verify()` -- We can use verify() to make sure a method is called. This can be combined with Mockito Matchers to make sure that the argument of the verified method is the correct type.

3. `ArgumentCaptor` -- We can use an ArgumentCaptor to inspect the argument of a verified method call.

The Mockito API documentation [can be found here](http://docs.mockito.googlecode.com/hg/org/mockito/Mockito.html). This is a fantastic resource.

Once we have our JUnit tests written, we are free to run them outside of an emulator or a device. In fact, we don't even need Robolectric since we have zero Android dependencies. Furthermore, our code is now completely decoupled from the Android framework -- meaning we have complete control over the behavior of our class (with exception of the Java standard library, of course).

If you find yourself with a method in your plain Java class that calls a method in another class that references an Android class, consider creating an interface between the two classes. Make the other class implement the interface, and then make your plain Java class require an instance of the interface -- not the class with the Android dependencies.

To run your JUnit tests, [follow the instructions here](http://blog.blundell-apps.com/android-gradle-app-with-robolectric-junit-tests/) for setting up Robolectric tests, but leave out the Robolectric. Running your tests should be as simple as running `./gradlew test` in your project's root directory.

**Your non-Android "dependencies" can be "injected" via good ol' constructors, Dagger injections, or my favorite combination -- calling the constructor of your class in the @Provides method of a Dagger scoped object graph. [Read here for more on that.](http://antonioleiva.com/dagger-3/)*