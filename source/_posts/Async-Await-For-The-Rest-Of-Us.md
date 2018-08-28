---
title: Async/Await For The Rest Of Us
tags:
  - async
  - async/await
  - await
  - csharp
  - C#
  - csharp async
  - C# async
categories:
  - 'Async/Await'
  - 'C#'
date: 2018-08-28 02:26:21
---

What's the deal with `async` and `await` in C#? Why should a .Net developer in 2018 **need** to know what it is and how to use it?

<!--more-->

Perhaps you've used `async/await` but find yourself having to go back and figure it out again? It's OK - I'm admittedly not an `async/await` guru either. 

I've had to figure out the hard way that, for example, using `ConfigureAwait(false).GetResult()` (as many might suggest) doesn't magically make your async method "work" synchronously.

But this isn't an in-depth look at the internals of `async/await` and how get fancy with `async/await`.

This is _"Async/await for the rest of us"_. Who are _the rest of us_?

We:

- Might have never used `async/await`
- Are not Microsoft gurus
- Are not C# gurus
- Admittedly forget how to use `async/await` properly at times. 
- Might have to go back to Google and figure out why a method can return a `Task` but not use the `async` keyword. 
- Might wonder why a method that's **not** marked as `async` can be awaited by it's caller? 

This - I hope - is an article for "the rest of us" that's to the point and practical.

_P.S. If you do want to dig into this topic more the best starting point is [I'd suggest starting with this article.](https://blog.stephencleary.com/2012/02/async-and-await.html)_

# The Async Keyword

There's confusion over the async keyword. Why? Because it looks like it **makes** your method asynchronous. But, it doesn't.

That's confusing. The `async` keyword **doesn't make my method asynchronous**? Yep.

## What Does it Do Then?

All the `async` keyword does is **enable the await keyword**. That's it. That's all. It does nothing else.

So, just think of the `async` keyword as the `enableAwait` keyword.

# The Await Keyword

The `await` keyword is where the magic happens. It basically says (to the reader of the code):

> I (the "thread") will make sure if something asynchronous happens under here, that I'll go do something else (like handle HTTP requests). Some "thread" in the future will come back here once the asynchronous stuff is done.

_Note: Notice that "thread" is in quotes. .Net abstracts **real** threads away from you. But that's beyond this article._

Generally, the most common usage of `await` is when you are doing IO - like getting results from a database query or getting contents from a file.

When you `await` a method that does IO, it's **not your app** that does the IO - it's ultimately the operating system. So your "thread" is just sitting there...waiting...

`await` will tell the current thread to just go away and do something useful. Let the operating system and the .Net framework get another thread later - whenever it needs one.

Consider this as a visual guide:

```c#
var result1 = await SomeAsyncIO1(); // OS is doing IO while "thread" will go do something else.
// A "thread" gets the results.

var result2 = await SomeAsyncIO2(result1); // "Thread" goes to do something else.
// One comes back...
await SomeAsyncIO3(result2); // Goes away again...
// Comes back to finish the method.
```

You might ask yourself at this point:

> If the `async` keyword doesn't make a method asynchronous then what does?

## What Makes a Method Asynchronous Then?

Well - it's not the `async` keyword as we learned. Go figure.

Any method that returns an "awaitable" - `Task` or `Task<T>` can be awaited using the `await` keyword. An "awaitable" method doesn't have to be an asynchronous method. But ignore that - this is meant to be for "the rest of us."

For the purpose of this article, we'll assume that "asynchronous method" is a method that returns `Task` or `Task<T>`.

### When Does This Happen?

When will we ever need to return a `Task` from a method? Again, it's usually when doing IO. Most IO libraries or built-in .Net APIs will have an "Async" version of a method. 

For example, the `SqlConnection` class has an `Open` method that will begin the connection. But, it also has an `OpenAsync` method. It also has an `ExecuteNonQueryAsync` method.

```c#
public async Task<int> GetSomeData() {
    using(var con = new SqlConnection(_connectionString))
    {
        // Some code to create an sql command "sqlCommand" would be here...
        await con.OpenAsync();
        return await sqlCommand.ExecuteNonQueryAsync();
    }
}
```

What makes the `OpenAsync` and `ExecuteNonQueryAsync` methods asynchronous is **not** the `async` keyword, but it is that they **return a `Task` or `Task<T>`**.

# Async All The Way Down

It is possible to do something like this (notice the lack of `async` and `await`):

```c#
public Task<int> GetSomeData() {
    return ThisReturnsATask();
}
```

And then `await` that method:

```c#
// Inside some other method....
await GetSomeData();
```

`GetSomeData` doesn't await - it just returns the `Task`. Remember that `await` doesn't care if a method is using the `async` keyword - **it just needs it to return a `Task`**. 

## It's A Best Practice

However, this is considered - generally speaking - a bad practice. Why? 

Since this article is supposed to be to the point and practical:

**Using async/await "all the way down" simply captures exceptions in asynchronous methods better.**

If you mark every method that returns a `Task` with the `async` keyword - which in turn enables the `await` keyword - it handles exceptions better and makes them understandable when looking at the exception's message and stack trace.

_Note: As always, there are more answers to this question beyond the scope of this article - like calling asynchronous code from synchronous code. P.s. That's a bad thing._

# Why Async?

What's the benefit of using `async/await`? 

## For Web Developers

If you build web apps using .Net technologies - the answer is simple: **Scalability**.

When you make IO calls - database queries, file reading, reading from HTTP requests, etc. - the thread that is handling the current HTTP request is **just waiting**.

That's it. **It's just waiting for a result to come back from the operating system.**

Performing a database query, for example, asks the operating system to connect to the database, send a message and get a message in return. But that is the OS making these requests - **not your app.**

**IO takes time.** Time where the waiting thread (in your app) could be used to do other stuff - especially **handling other HTTP requests**.

> Using `async/await` allows your .Net web apps to be able to handle **more HTTP requests while waiting for IO to complete.**

## For Desktop/App Developers

But desktop apps don't handle HTTP requests...

Well, desktop apps **do** handle user input (keyboard, mouse, etc.), animations, etc. And there's only **one UI thread** to do that.

### User Input Is User Input

If we consider that HTTP requests in a web app are _just user input_, and desktop (or app) keyboard and mouse events are _just user input_ - then it's actually worse for desktop/app developers! They only get **one thread** to handle user input!

The IO issues and fixes still apply.

However, the issue of CPU intensive tasks is another concern. In a nutshell, these types of tasks should not be done on the UI thread.

The types of tasks would include:

- Processing a large number of items in a loop
- Computing aggregations for reporting

 If your app does this (on the main/UI thread), then there's nothing to handle user input and UI stuff like animations, etc.

This leads to freezing and laggy apps.

The solution is to offload CPU intensive tasks to a background task/thread. This starts to get into queuing up new threads/tasks, using `ConfigureAwait(false)`, etc. Things beyond the scope of our article.

# Conclusion

To summarize briefly:

- The `async` keyword doesn't make methods asynchronous - it simply enables the `await` keyword. 

- If a method returns a `Task` or `Task<T>` then it can be used by the `await` keyword to manage the asynchronous details of our code.

- Doing IO always results in blocking threads. This means your web apps can't process as many HTTP requests in parallel and freezing and laggy apps.

- Using `async/await` helps us create code that will allow our "threads" to stop blocking and do useful work while performing IO.

- This leads to web apps that can handle more requests per second and apps that are more responsive for their users.

There's so much more to be said and so many more concepts surrounding `async/await`. Some include:

- What is a `SynchronizationContext`? When should I be aware of this?
- Why can I mark a method as `async void`? What does this do? Should I do this?
- How do I offload CPU intensive work to a background thread/task?
- Is it possible to do work on a background thread and return to the UI thread at the end?
- How do I call a method marked with the `async` keyword from synchronous code? What happens when I do this?

**Let me know what you think - or if I've missed something etc. Thanks!**

# P.S

Tired of endless configuration and infrastructural setup in your .Net Core apps? Not sure where to start? I've been building a tool to make pieces like Task Scheduling, Queuing, Caching, Mailing, etc. a breeze!. Take a look and let me know what you think - [it's called Coravel](https://github.com/jamesmh/coravel).

Don't forget to connect with me on [twitter](https://twitter.com/jamesmh_dev) or [LinkedIn](https://www.linkedin.com/in/jamesmhickey/)!








