---
title: 'Functional Programming With C#: Simple Use Case'
tags:
  - .net
  - 'c#'
  - clean code
  - functional programming
  - 'functional programming c#'
  - refactoring
url: 298.html
id: 298
categories:
  - 'C#'
  - Functional Programming
  - Software Architecture And Design
date: 2017-08-25 03:04:04
---

[**Update**: I created a YouTube video based on this post, if you are interested....](https://youtu.be/dfwBEIr5giY) During the past year and a half I have been making a point to learn about and develop my skills in functional programming. For the majority of this experience, I have been using JavaScript and [TypeScript](http://www.typescriptlang.org/). After moving onto another project at work, I was brought back to "C# and SQL world." Long story short, I noticed the way I was thinking about software problems in C# were totally different than before.

<!--more-->

### A Twisted Mind

I found myself using functional programming techniques in C# - combined with language features like [generics](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/) \- just "out-of-box", because my brain was already wired this way due to my experiences in the past year or so. And not because it makes "fancy" code, but because it makes my code more readable, maintainable and flexible. Especially when refactoring duplicated code - but that's another story (perhaps for a future post).

### Blast From The Past

I have a simple use case which comes from what I've been working on at my day job (if you want to know, I've been building some stateless web services using [Microsoft's Web API](https://www.asp.net/web-api)). I had created a "StopWatch" class (probably) a year and a half ago. In building the new APIs, I needed to time certain parts of the system - to see what was "taking so long". Mr. StopWatch and I became good acquaintances again. At this point, the StopWatch was used like this:

    StopWatch.Start();
    
    /* Some code that did stuff */
    
    StopWatch.Stop();
    StopWatch.LogEllapsedMs("This code took {0} ms to execute.");
    

Not bad for "Year and a half ago James". But having to do this for every single piece of code I wanted to test separately... it was really looking like this:

    StopWatch.Start();
    DoThisOneThing();
    StopWatch.Stop();
    StopWatch.LogEllapsedMs("DoThisOneThing() took {0} ms to execute.");
    
    /* Repeat that code above like 6 times... */
    

Nevermind the fact that this doesn't allow for nested logging... To log the performance of one line of code it turns into 4 lines? Fine. But it's just mentally taxing - **even when I understand what's going on.**

### A Note On Clean Code

I was watching a talk on YouTube the other day, and the biggest piece of knowledge I gained was about how people read - whether code, or books, etc. This gentleman said something like "Take every line of code and turn it into "X"s. Can you understand what pieces belong together?" So, my code above would look like this:

    XXXXXXXXXXXXX
    XXXXXXXXXXXX
    XXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXX
    /* Code repeated again */
    XXXXXXXXXXXXX
    XXXXXXXXXXXX
    XXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXX
    

Can you make out what parts belong together and which parts don't? Nope. Keep reading...This will come back to haunt you.

### Functional Programming In C# To The Rescue

So, the solution for my code was really simple - and, as you can guess, involves using functional programming as the answer. I don't think I need to explain it. Once you see it you will get it.

    StopWatch.LogEllapsedMs("This thing took {0} ms", () => {
         DoSomeStuff();
    });
    

Now I can do stuff like this:

    StopWatch.LogEllapsedMs("This entire section of code took {0} ms", () => {
         DoSomeStuff();
    
         StopWatch.LogEllapsedMs("This sub portion of code took {0} ms", () => {
               ThisOtherThing();
               SomeOtherStuff();
    
               StopWatch.LogEllapsedMs("This one piece took took {0} ms", () => 
                    SomeOtherThing());
         });
    });
    

Imagine trying to have nested performance logging with the way I was doing it before? Well...since I was using a static utility class, it just wasn't possible. Now watch this, using C# generics I am able to return the results of whatever is being logged:

    public Errors CheckForErrors() {
         return StopWatch.LogEllapsedMs("This thing took {0} ms", () => 
              DoSomeStuffAndReturnErrors());
    }
    

I can even log a function that returns a value in one-line! That's super cool. And, it's easy to read. Take the example with the nested logging. Now replace that code with "X"s. Here...I'll do it for you:

    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
        XXXXXXXXXXX
    
        XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
               XXXXXXXXXXXXXXXXXXX
               XXXXXXXXXXXXXXXXXXX
    
               XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
                    XXXXXXXXXXXXXXXX
    

### No More Brain Pain

You don't know what that block of "X"s is doing (because it just "X"s...), but you can tell what parts of the code belong together due to the indentation. Here's the point, **This is what my brain sees when I am scanning code.** This makes my brain hurt less when reading code. This makes everyone's head hurt less when reading my code.

### Wanna See The Code?

Well... you asked for it. Here it is in all it's glory.

    public class StopWatch
    {
        public static T LogEllapsedMs<T>(string formatMessage, Func<T> action)
        {
            Stopwatch watch = new Stopwatch();
            watch.Start();
            T result = action();
            Debug.WriteLine(string.Format(formatMessage, watch.ElapsedMilliseconds));
            return result;
        }
    
        public static void LogEllapsedMs(string formatMessage, Action action)
        {
            Stopwatch watch = new Stopwatch();
            watch.Start();
            action();
            Debug.WriteLine(string.Format(formatMessage, watch.ElapsedMilliseconds));
        }
    }
    

Not very impressive, eh? By using a simple functional programming technique, all the code using this performance logging is now: 1. Way easier to read 2. Allows nested logging 3. Allows logging code and returning a result **without having to use temporary variables** many times That's the beauty of using functional programming techniques, they don't have to be complicated like all the scientific and math gurus out there try to tell you. Just start by using a function that expects another function. Just that alone can change your thinking in huge ways. And this is just the beginning...

If You Enjoyed This...
======================

I've been writing about functional programming lately - here are some selected posts you may enjoy:

*   [Advanced Fluent Interfaces: LINQ Case Study](https://www.blog.jamesmichaelhickey.com/advanced-fluent-interfaces-linq-case-study/)
*   [3 Benefits Of Fluent Interfaces](https://www.blog.jamesmichaelhickey.com/3-benefits-fluent-interfaces/)
*   [Exploring Fluent Interfaces](https://www.blog.jamesmichaelhickey.com/exploring-fluent-interface/)

**Let me know what you think - and don't forget to subscribe below if you enjoy this kind of content!**