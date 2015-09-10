---
layout: post
title: "Notes on Updating to Retrofit 2"
date: 2015-09-09
---
Square recently released a beta version of Retrofit 2. It's a major update, well deserving of its version bump (Android 6.0 anyone?) complete with an overhauled API, a required dependency on OkHttp, and some breaking changes. Jake Wharton [gave a talk](https://www.youtube.com/watch?v=KIAoQbAu3eA) on it at Droidcon NYC and documentation updates are coming soon. As such, I won't attempt to provide an exhaustive list of the changes. I'll just give a quick overview of what I encountered in upgrading from version 1 to version 2.

**OkHttp**

Retrofit 2 relies heavily on OkHttp, to the effect that OkHttp is now a required dependency for projects using Retrofit 2. This means no more `Client` interface for plugging in a custom HTTP client like Volley, HttpUrlConnection, or some custom implementation that loads JSON files from disk. Retrofit 2 still uses the builder pattern for service interface instantiation, but the required argument for `Retrofit.Builder.client()` (no more `RestAdapter`) is now an instance of `OkClient`.

If you're not using OkHttp already, this might be a bit of pain point in upgrading. But if you haven't already spent a ton of effort in customizing an HTTP client, the increased reliance on OkHttp allows Retrofit to be less general in its treatment of HTTP requests and instead leverage OkHttp's already robust API.

For example, getting the value of a specific header from a Retrofit `Response` used to entail calling `Response.getHeaders()` which returned a `List` of Retrofit's `Header` objects, iterating through this `List`, checking for a matching key, then getting the value corresponding to that key. Now, `Response.headers()` returns a `Headers` object from OkHttp. Getting a specific header value is now as simple has `response.headers().get("some_key")`.

**More OkHttp -- Interceptors!**

[Interceptors](https://github.com/square/okhttp/wiki/Interceptors) are one of OkHttp's more powerful features. Now that Retrofit 2 explicitly depends on OkHttp, Retrofit's previous interceptor-like functionality is gone -- leaving the user to implement any previous functionality with OkHttp interceptors instead.

The most obvious example in Retrofit 2 is the disappareance of the `RequestInterceptor` interface. Instead of creating an implementation of Retrofit's `RequestInterceptor`, we can now duplicate or extend any of previous functionality in an OkHttp `Interceptor`.

Another example is the disappareance of Retrofit's `Log` interface. I understand that logging in some form *might* be added at a later date (Retrofit 2 is still in beta). Anyway, if you want logging, you can easily create an `Interceptor` to do this. If you look at the [OkHttp wiki page](https://github.com/square/okhttp/wiki/Interceptors), there's actually some example code for a `LoggingInterceptor`.

An obvious difference in the API of Retrofit 1 and Retrofit 2 is that the these interceptors are now added in the creation of an `OkClient` -- not the service interface created with `Retrofit.Builder` (previously `RestAdapter.Builder`).

**Goodbye RetrofitError**

`RetrofitError` is gone. This is probably a good thing -- it was always a [bit of a pain](https://github.com/square/retrofit/issues/284) to work with.

However, the current API is still a bit complicated with respect to error handling. If our API requests are defined to return a `Response` object, then requests can still throw an `IOException` in the event of a timeout or a network error (i.e. airplane mode errors). But if the request completes with a non-successfull HTTP response code (outside of the 200 - 299 range), then we have to call `Response.isSuccess()` to check if the response completed with a successful error code, while also handling the `IOException` case.

If we don't want to touch `IOException` directly, we can define our requests to return a `Result` type (both `Response` and `Result` use generic type arguments for the response body). If the request throws an exception, then `Result.isError()` returns true. But if the requests completes with a non 200 to 299 error code, `Result.isError()` return false.

The upside of all this is clear when working with RxJava, especially in the case where you don't want non-200 responses to close a stream. Because completed HTTP requests that return non-200 codes don't throw, a subscriber's `onError()` isn't called, and the stream survives. This puts the bulk of our error-handling code into `onNext()`, although you may still want to implement `onError()` for `IOException`s and the like.

**`Call` me maybe**

Perhaps the biggest change in Retrofit 2's API is the addition of the `Call` interface -- the return type that most people will be using if they aren't using RxJava or some custom return type (defined in a `CallAdapterFactory` -- also new).

A `Call` encapsulates both the request and response. If we define a request method to return a `Call` object, then we can call `Call.execute()` to run a request synchronously, which returns a `Response`. We can also use `Call.enqueue()` with a `Callback` argument to run a request asynchronously. This is quite a bit different from the Retrofit 1 API, where a call on a service interface method was equivalent to an invocation of the HTTP request itself.

For example, we may have previously used `GithubService.repositories()` to synchronously return a `List<Repository>`. Now, we can define `GitHubService.repositories()` to return a `Call<List<Repository>>`, then execute the request with `Call.execute()` or `Call.enqueue()` at a later time or from a different location in our code!

**Impressions**

`OkHttp` is a super powerful HTTP client library, and Retrofit 2 seems to work *with* it instead of just *around* it. After all, Retrofit is *not* an HTTP client, and HTTP client-like behavior, like interceptors, *should* be in OkHttp -- not Retrofit.

Furthermore, the addition of the `Response`, `Result`, and `Call` types (among others) and the removal of `RetrofitError` are tangible improvements to the core Retrofit API.

The result is a leaner, more focused, and ultimately more mature version of Retrofit.