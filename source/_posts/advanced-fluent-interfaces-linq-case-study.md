---
title: "Advanced Fluent Interfaces: LINQ Case Study"
tags:
  - "c#"
  - fluent api
  - fluent interface
  - functional programming
url: 421.html
id: 421
categories:
  - "C#"
  - Fluent Interfaces
  - Functional Programming
  - Software Architecture And Design
date: 2017-11-01 19:03:45
---

In Martin Fowler's ["famous" article about fluent interfaces](https://www.martinfowler.com/bliki/FluentInterface.html), he talks about how it's beneficial (when using strongly typed languages) to have the return type of your fluent method be flexible. We'll be looking at this advanced fluent interface technique today.<!--more-->

# Why?

Some benefits of building your fluent objects this way are:

- **Restrict** differing paths of logic.
- **Enable** differing paths of logic.
- **Better IDE code completion** (as Fowler mentions in his article above - this makes your fluent objects more like "a wizard in the IDE")

# An Example Of Advanced Fluent Interfaces From C#'s LINQ

I want to use a simple example that most C# / .NET developers will be familiar with - from LINQ.

Whenever you call the `OrderBy` method in LINQ, that method will actually give you **new** methods that you are able to call (e.g. `ThenBy` and `ThenByDescending`).

```
someIEnumerable
     .OrderBy(item => item.SomeProp) // This gives access to "ThenBy()"
     .ThenBy(item => item.SomeOtherProp);
```

It doesn't make sense to perform `ThenBy` on it's own. It only makes sense when you are currently ordering the elements in your collection, and you **further** want to order.

# How Does This Happen!?

The way to perform this is very simple - and flexible. Anytime you want your fluent object to return an "alternate path" (i.e. restrict or add methods that are accessible to the caller) you just return the fluent object casted to an interface (which includes or excludes methods you require).

For example, LINQ methods (normally) return an object using the `IEnumerable` interface. But when you call `OrderBy`, you get an `IOrderedEnumerable` object.

The `IOrderedEnumerable` interface inherits from `IEnumerable` - so it has all the methods you normally would have access to. But - it also defines the extra `ThenBy` and `ThenByDescending` methods.

# Next Time... Maybe

I'd like to post sometime soon about creating our own "polymorphic" fluent objects and see how powerful they can become.

# If You Enjoyed This...

I've been writing about Fluent Interfaces and Functional Programming lately - here are some related posts:

- [Functional Programming In C# - A Simple Use Case](https://www.blog.jamesmichaelhickey.com/csharp-functional-programming-a-simple-use-case/)
- [3 Benefits Of Fluent Interfaces](https://www.blog.jamesmichaelhickey.com/3-benefits-fluent-interfaces/)
- [Exploring Fluent Interfaces](https://www.blog.jamesmichaelhickey.com/exploring-fluent-interface/)

# Keep In Touch

Don't forget to connect with me on [twitter](https://twitter.com/jamesmh_dev) or [LinkedIn](https://www.linkedin.com/in/jamesmhickey/)!

<div style="padding:0   20px; border-radius:6px; background-color: #efefef; margin-bottom:50px; margin-top:20px">
    <h1 class="margin-bottom:0"> Navigating Your Software Development Career
</h1>
An e-mail newsletter where I'll answer subscriber questions and offer advice around topics like:

✔ What are the general stages of a software developer?
✔ How do I know which stage I'm at? How do I get to the next stage?
✔ What is a tech leader and how do I become one?


<div class="text-center">
    <a href="http://eepurl.com/gdIV5X">
        <button class="btn btn-sign-up" style="margin-top:0;margin-bottom:0">Join The Community!</button>
    </a>
</div>
</div>
