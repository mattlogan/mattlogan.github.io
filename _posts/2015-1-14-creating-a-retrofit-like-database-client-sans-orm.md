---
layout: post
title: "Creating a Retrofit-like Database Client sans ORM"
date: 2015-1-14
categories: android architecture retrofit architecture
---
[Retrofit](http://square.github.io/retrofit/) for Android allows us to define a REST client as an interface and create instances without having to provide an implementation. This is great because:

1. The REST client implementation is decoupled from our application code.
2. We don't have to write the REST client implementation.

We can also achieve **database client decoupling** (a win on number one) if we're willing to provide our own implementation (forego number two), which really isn't all that bad.

Developers often look for Android ORM libraries because writing a database implementation is scary. However, the consequence is often a tight coupling between application code and database implementation. For example, several ORM libraries require that your model classes, or POJOs, subclass a base model class that facilitates the conversion from Java object to database item. As a result, your model classes become tightly coupled to the library's database implementation (we cannot even call them POJOs anymore at this point).

The alternative is to write your own implementation. The Android framework makes this pretty easy. As a quick primer, the Android class `SQLiteOpenHelper` allows us to simply control database creation and migration, and also gives us access to an instance of `SQLiteDatabase`. A `SQLiteDatabase` instance provides simple database CRUD methods, and we can also execute raw SQL statements with `execSql()`. Online tutorials abound!

As an example, let's say we're creating a database client for saving and loading `Dog` objects. We can create a Retrofit-like interface:

{% highlight java %}
public interface DogDatabaseClient {
    public void saveDogs(List<Dog> dogs);
    public void loadDogs(DogsCallback callback);
}
{% endhighlight %}

In the spirit of learning (and laziness), I'll leave the implementation details up to the reader, but it would look something like this:

{% highlight java %}
public class SqlLiteDogDatabase implements DogDatabaseClient {

    // Our implementation of SqlLiteOpenHelper
    private final DatabaseHelper dbHelper;    

    public SqlLiteDogDatabase(DatabaseHelper dbHelper) {
        this.dbHelper = dbHelper;
    }

    public void saveDogs(List<Dog> dogs) {
        // Use dbHelper instance to get db instance and save dogs
    }

    public void loadDogs(DogsCallback callback) {
        // Use dbHelper instance to get db instance and load dogs
        callback.onDogsLoaded(dogList);
    }
}
{% endhighlight %}

Then, wherever an instance of `DogDatabaseClient` is required, we supply an instance of `SqlLiteDogDatabase`. With Dagger, for example, we'd have a `@Provides` method in our data `Module` like this:

{% highlight java %}
@Provides @Singleton
DogDatabaseClient provideDogDatabaseClient() {
    return new SqlLiteDogDatabase(new DatabaseHelper());
}
{% endhighlight %}

The result, if done correctly, is a database client interface that can be referenced from our application code without any direct coupling to the database client's implementation. The benefits of programming to an interface instead of an implementation are discussed heavily elsewhere, so I'll avoid that discussion here.

You don't need to use an ORM library that turns your application code into something else. Instead, you can create your own database client interface and its implementation and reap the benefits of good, clean code.