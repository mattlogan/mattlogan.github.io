I recently wrote a small library called [Pancakes](https://github.com/mattlogan/pancakes). It's intended to be a replacement for FragmentManager for projects that use Views in place of Fragments. The motivation for not using Fragments is covered in depth in a [blog post](https://corner.squareup.com/2014/10/advocating-against-android-fragments.html) by Square, but it boils down to this: Fragments are overcomplicated, difficult to debug, and mostly unnecessary. Unless you're using Loaders, Fragments only to serve to a) manipulate Views and b) be managed by the FragmentManager to maintain a navigation stack. Manipulating Views can be done from Views themselves, a presenter, or really anything else.

The goal for this library is to address the latter concern -- managing navigation of a View stack -- and the purpose of this blog post is to explain the decisions I made in designing and implementing this library.

**A navigation stack should look like a Stack**

Since app navigation very closely resembles a stack of Views, I decided on a single class called ViewStack that would have public methods very similar to Java's Stack data structure -- push, pop, peek, etc. In its implementation, ViewStack actually uses a Stack. ViewStack is essentially a proxy for this internal Stack with some extra functionality added on.

**Going to work at the ViewFactory**

My first attempt for ViewStack's internal Stack involved an actual Stack of Views, i.e. a `Stack<View>`. This was problematic for two reasons. First, this keeps all Views in memory, which was not my intention. Second, it would be impossible to maintain the Stack across configuration changes without leaking the contained Views and their  Contexts.

I needed a generic type that would allow for deferred View creation without maintaining any references to Contexts. I settled on an interface I called `ViewFactory` with one method -- `createView(Context)`. Because a Context can be passed as an argument to the creation method, a ViewFactory instance doesn't need to hold a Context reference. Furthermore, I made ViewFactory implement Serializable such that a `Stack<ViewFactory>` could be serialized in a Bundle and persisted through configuration changes.

**Bundle it up**

I needed some mechanism for putting the relevant state of a ViewStack in a Bundle without persisting the entire ViewStack, since it contains a reference to the supplied ViewGroup container. Since Stack is serializable and ViewFactory is also Serializable, I figured that putting ViewStack's internal `Stack<ViewFactory>` into a Bundle was my best bet. However, I didn't want to expose the `Stack<ViewFactory>` instance via a public accessor, because I consider this instance an implementation detail (i.e. not part of ViewStack's public API).

Instead of exposing the internal `Stack<ViewFactory>`, I just added two methods to ViewStack: `saveToBundle(Bundle)` and `rebuildFromBundle(Bundle)`. There's a drawback here -- the user is trusting that ViewStack doesn't mess with the Bundle in any other way. But this seems like a worthy tradeoff to keep its implementation hidden.

**A ViewStackDelegate to finish() the job**

When I was playing around with this library in its development, I couldn't quite figure out the best way to deal with an empty stack. In the end, I went with the simplest solution, though it may not be the most extensible. The tricky decision is figuring out what to do in the case where `pop()` is called when the size of the internal stack is one.

If we were to continue as usual, this would empty the container of all Views, leaving a presumably empty container on the screen. Then I had the idea of having the user of the library supply a delegate when creating the ViewStack to offload this decision to a user-supplied delegate implementation. There's still a question of how much control to give to the library user. For example, should the delegate get to decide if the last View should be popped from stack? Should the delegate be notified when the Stack is then empty, or even on every stack change?

In the end, I went with a solution that seems to fit the most common use case. When `pop()` is called when the internal stack size is one, this makes a call to `ViewStackDelegate.finishStack()`, which can then decide how to proceed. The `pop()` call does not pop the last ViewFactory off the internal stack, and it does not remove the last View from the container. I expect that `finishStack()` should call finish() in the Activity that hosts the Views, but I left this up to the delegate to implement.

**Preconditions**

ViewStack throws lots of unchecked exceptions for programming errors -- for example, supplying a null Bundle argument to `saveToBundle(Bundle)`. I got the idea of using a Preconditions class from Jake Wharton's [u2020](https://github.com/JakeWharton/u2020/blob/master/src/main/java/com/jakewharton/u2020/util/Preconditions.java) project, though I think the idea of "aggressive programming" (as opposed to "defensive programming") is much older.

The intention is to fail as early as possible to remove the possibility of unexpectedly failing at a later time due to bad arguments or incorrect method calls. Furthermore, the user of the library shouldn't see any exceptions thrown by ViewStack's implementation details. If `saveToBundle(Bundle)` is called with a null argument, we don't want `Bundle.putSerializable(Serializable)` to crash, because that's an implementation detail. Instead, we throw immediately to let the user know of the error as close to the point of failure as possible (supplying a null argument).

**Debugging with LeakCanary, i.e. rotating my phone until something bad happens**

I used LeakCanary a lot in development. While there are unit tests in the library, I wanted to make sure it worked in practice. Of course, when dealing with Contexts, Views, and configuration changes (namely device rotation), we need to be super careful about memory leaks. A big part of testing and debugging involved writing a sample app with a stack of Views, navigating back and forth between them, and rotating the device a bunch of times until LeakCanary did or didn't complain!
