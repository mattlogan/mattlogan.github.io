---
layout: post
title: "Android Can Have Delegates, Too"
date: 2014-11-19
categories: android delegate architecture
---
The Cocoa framework for iOS makes heavy use of Delegates: objects that implement a Protocol and can be used to receive, return, or manipulate information.

While the Android framework has no such naming convention, the pattern is absolutely allowed. In fact, it's less a property of the Cocoa framework and more a property of object-oriented programming in general.

The book *Design Patterns* describes delegation as follows:

> In delegation, *two* objects are involved in handling a request: a receiving object delegates operations to its **delegate** . . . For example, instead of making class Window a subclass of Rectangle (because windows happen to be rectangular), the Window class might reuse the behavior of Rectangle by keeping a Rectangle instance variable and *delegating* Rectangle-specific behavior to it. In other words, instead of a Window *being* a Rectangle, it would *have* a Rectangle. Window must now forward requests to its Rectangle instance explicitly, whereas before it would have inherited those operations.

In Android-land, I've found myself creating abstract base classes that perform some task with the idea that I'll need that exact functionality to be performed in several subclasses at a later time. Requirements inevitably change; we then have some subclass that needs slightly different functionality from what the superclass provides. This leaves me in a tough spot.

If we follow the path of delegation, we can put the common functionality in a delegate class that includes hooks to alter its behavior. Or we can have two separate delegate classes that implement a common interface and can be swapped in and out at runtime! We have quite a bit more flexibility here.

Finally, delegation is a looser coupling mechanism than inheritance. While this may lead to code that is slightly less clean or readable, it can mean more flexibility when used intentionally and carefully.