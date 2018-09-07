---
title: Keeping Your ADO Sql Connections Safe
tags:
  - ado
  - ado connection
  - database connection
  - sqlconnection
url: 666.html
id: 666
categories:
  - ADO
  - "C#"
  - Functional Programming
  - Refactoring
date: 2018-08-15 22:00:27
---

What happens when you don't close your .NET `SqlConnection`s? Bad stuff. Bad stuff that will inevitably bring your IIS website crumbling down to ashes. Well... something like that. We all know that we should take extra care to always **close** our DB connections. Right?

But what happens when developers try to get fancy with their DB connections and focus more on being able to re-use open DB connections rather than being safe? Well, the consequences were bestowed upon me a few weeks ago.

Let me explain (briefly) what happened, what _could_ happen to you, and a way to fix it - while maintaining the flexibility of re-using open DB connections and being able to safely use DB transactions from your C# code (using ADO).

<!--more-->

# When Leaky Connections Strike

In the codebase I work with for my day job, there are utilities for performing SQL queries (well... calling stored procs) in C#. When I first saw this code (many months ago) I immediately thought that it was not a good thing (and told the team my thoughts).

Why? Because mishandling DB connections lead to very bad things. Better to be safe than sorry. But, it wasn't considered an issue - it never had caused any issues yet. It was working well so far. So, it wasn't worth fussing over - other than making my concerns public.

The pattern in question is something like this:

1. A method that accepts a `SqlConnection` as a parameter.
2. Checks if the `SqlConnection` is already opened. If it is, don't close it since the caller wants to keep it open to issue further queries.
3. If the `SqlConnection` is closed, open it, then run the query, then close it.

The code might look like this:

```
if (cmd.Connection.State != ConnectionState.Open)
{
    cmd.Connection.Open();
}

// Do your stuff...

if (origConnectionState == ConnectionState.Closed)
{
    cmd.Connection.Close();
}
```

And yes, there is no try block.

# When It All Comes Crumbling Down

A few weeks ago, we had major issues where our site kept locking up for everyone. Long story short - the code was calling a stored procedure that didn't exist, the DB call threw an error, and the calling code (just by chance!) silently caught the error and kept on chugging away.

Since the code was attempting to be fancy with the connections - when an error was thrown it by-passed the code that was supposed to close the connection. Normally, exceptions like this would be caught during development (and fixed).

But what happens when things aren't thoroughly tested? And the caller is silently catching the error? Boom!

Users saw no error. Until all those un-closed connections piled up.

This took a full day to figure out where the issue was coming from since the exception was silently caught! Eventually, looking at IIS logs and the event viewer gave us some clues. I literally found and fixed the issue during my shift's last ten minutes. Phew! What a scramble!

# Why Was This Done?

Why would anyone do this? Well, the reasoning is that the caller of this method might want to issue multiple queries. Instead of having to open and close multiple `SqlConnection`s, we can have more performance by keeping one opened and closing it ourselves after all our queries are completed. Avoiding the extra re-connections to the database would be avoided.

In reality, there was not even _one_ instance where a caller was re-using an open `SqlConnection` in the entire codebase. :(

# But .NET Handles ADO Connections For You!

Is it true that this _would_ boost performance though? Not really. By default, .NET ADO connections are [pooled](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/sql-server-connection-pooling). This means that calling `Close()` on your DB connection doesn't _really_ close it (physically).

.NET handles this under the covers and keeps a pool of active connections. Any performance benefit gained is most likely just the mere cost of instantiating a new c# object in memory and the cost of finding a connection in the pool. The **actual hit** of connecting to the database is most likely not gained.

> Rule of thumb: Test real-world scenarios if performance is an issue. Then, test your optimizations under those same scenarios and compare.

# Who's Responsibility Is It?

There's another issue here. Who is responsible for managing the "low-level" database access? In the code above - it's:

1. The low-level database utilities
2. And anyone who wants to use them!

So, essentially, you could have an MVC controller who is creating a `SqlConnection`, passing it into the utility to perform a query, then closing the connection back inside the controller action.

But that never happens, right? Well, sometimes we inherit code that does do this. :(

> This is like going to a mechanic's shop, asking them to fix your car, but giving them **your cheap tools to use to fix the vehicle**.

The same is true with the code above. Lower-level database objects should always own that domain/responsibility. Someone else shouldn't be telling them how to manage their database connections.

# What To Do. What To Do.

How could we _safely_ re-use open `SqlConnection`s if we wanted to?

In this same code base, I introduced using [Dapper](https://github.com/StackExchange/Dapper) to replace database access moving forward. Dapper turned out to be more than 5 times faster than the hand-built mini-object mapper that was in place anyways. That's another story for another day.

In conjunction with these changes, I had created a few methods that allowed us to use `SqlConnection`s safely - but retaining the flexibility of connection reuse and the ability to use transactions.

In [a past post](https://www.blog.jamesmichaelhickey.com/csharp-functional-programming-a-simple-use-case/) I explained how you can use higher-order functions to encapsulate repetitive code while allowing the "meat" of your code to be passed in. What do I mean?

Take, for example, a `Query`class that has the method `UsingConnectionAsync` which might be used like this:

```
await _query.UsingConnectionAsync<int>(
    async con =>
    {
        var result1 = await con.ExecuteAsync(sql1, parameters1); // ExecuteAsync comes from Dapper
        var result2 = await con.ExecuteAsync(sql2, parameters2);

        return PutTheResultsTogether(result1, result2);
    }
);
```

Such a method accepts a function that will be given a `SqlConnection`. The function can use that `SqlConnection` however it wants. Once done, the higher-order method `UsingConnectionAsync` will manage to close the connection "magically" (in a manner of speaking).

We can wrap all our database related code inside of a "scope", as it were, and just do what we want. It's safe and yet flexible.

You could make another method `UsingTransactionAsync` that would manage to open a `SqlConnection`, start a transaction, and rollback on errors while committing successful usages.

[See the section below](#gist) for a gist containing one way to implement these methods.

# Conclusion

Hopefully, this real-world crisis will encourage you to think about how your `SqlConnection` objects are being managed. Especially in older "legacy" applications (ya, we all have to deal with them), you might see these kinds of attempts at optimizing database connections.

Just remember:

- Be safe and keep low-level responsibilities in one place. DB connection management and query logic are not the same things.
- Even if the code is obviously not causing immediate issues - if it looks stinky, it might be stinky. Better safe than sorry.
- Higher-order functions are a great way to encapsulate responsibilities of one domain/object while allowing the callers to still be flexible in how they can use those managed resources.

# Keep In Touch

Don't forget to connect with me on [twitter](https://twitter.com/jamesmh_dev) or [LinkedIn](https://www.linkedin.com/in/jamesmhickey/)!

I also have an e-mail letter where I'll give you tips, stories and links to **help you get to the next step of your career as a software developer**. I'll also give you updates about stuff that I've been working on ;)

[Subscribe if you haven't already!](https://tinyletter.com/jamesmh)

# P.S.

I've been building tools for indie .NET Core developers needing to get their next groundbreaking app or side-project to market faster - without compromising code quality and elegance. [It's called Coravel!](https://github.com/jamesmh/coravel). Check it out and let me know what you think ;)

# <a name="gist"></a> Extras: Gist

<script src="https://gist.github.com/jamesmh/9e1382a567e0891670c2d55b47ec3ba7.js"></script>
