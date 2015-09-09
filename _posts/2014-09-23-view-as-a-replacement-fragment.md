---
layout: post
title: "View as a Replacement for Fragment"
date: 2014-09-23
categories: android view fragment
---
Reusability of user interface components in mobile app development is a nice goal, and can be achieved in a variety of ways. The most common approach involves the Fragment, while a less common (but still great) approach involves the simple View.

The classic example of reusability is the landscape tablet view. It's smart to design UI components (vagueness of the word "component" is intentional here) in such a way that they will look good in normal single-pane viewing and in two-pane viewing on a tablet in landscape orientation.

Our goal as Android developers is to write modular code such that a single complete class can be inserted anywhere with little extra work.

Specifically, we'd like to be able to add a View or collection of Views with some dynamic behavior into an existing View tree.

![View Tree](https://raw.githubusercontent.com/mattlogan/mattlogan.github.io/master/assets/viewtree.png)

The typical solution is the Fragment. The Fragments API Guide explains:

> You can think of a fragment as a modular section of an activity, which has its own lifecycle, receives its own input events, and which you can add or remove while the activity is running (sort of like a "sub activity" that you can reuse in different activities).

Fragments favor [composition over inheritance](http://en.wikipedia.org/wiki/Composition_over_inheritance). A Fragment is not itself a View, but rather a composition of Views (in its most common use case). This theoretically offers the developer some added flexibility, but I'm not sure if that's important in this case.

Additionally, Fragments have their own life cycles, though they are closely tied to the life cycles of their host Activities. This makes some things a bit more convenient -- for example, offering the developer a clearly defined place to set up and tear down View and model objects.

However, Fragments are unnecessary in what I believe to be their most common use case -- adding a view layout and its associated behavior to another layout.

For this use case, our alternative is to subclass View or one of its subclasses and use this class as our modular, reusable UI component.

In Fragments, we can perform object creation and configuration in onCreate(), onCreateView(), or onResume(). In Views, we can move this work to the View constructor, onAttachedToWindow(), or onFinishInflate(). We also have onDetachedFromWindow() as a replacement for onPause() or onDestroy().

Furthermore, we can create public methods in our custom View class to be called from the life cycle methods of the Activity in which it exists -- effectively binding the View and its instance variables to the Activity life cycle.

In Fragments, we can inflate a View from XML and return it in onCreateView(). In a custom View, we can also inflate a View from XML and add it with addView() if our custom View extends ViewGroup. To avoid an extra ViewGroup at the root of the inflated View, we can use the merge tag as its root element.

The result is a lightweight, modular, reusable UI component that lacks some of the heavier functionality of Fragments, but can probably still serve most use cases.