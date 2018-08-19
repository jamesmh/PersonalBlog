---
title: 3 Benefits Of Fluent Interfaces
tags:
  - design patterns
  - fluent api
  - fluent interfaces
  - software design
url: 345.html
id: 345
categories:
  - 'C#'
  - Fluent Interfaces
  - Object Oriented Programming
  - Software Architecture And Design
date: 2017-09-06 10:49:28
---

In my [previous post](https://www.blog.jamesmichaelhickey.com/exploring-fluent-interface/) I looked at a few reasons that I've seen given for thinking that fluent interfaces are "not all that." I personally think that fluent interfaces can add some serious benefits to developers.

<!--more-->

### All Reusable Code Is An API

I've heard it said that internal code (that is, code which is expected to remain in the project's codebase and not be exposed otherwise) should not be treated in the same manner as code which will be exposed (via an API, library, etc.). I've also heard the opposite said - all reusable code is an API (for other developers on your team, at the very least). In my experience, I have found the tools I enjoy using the most are those which have a very clean and readable API which conveys the meaning of what the code is actually trying to do. And, **developers I trust and value the most are those who write their code as-if someone, in a future yet unseen, may need to figure out how to use it...** That being said, one of the traits of good software code I stand-on is that your code needs to have meaning and be easily readable. I don't care if you are using a `foreach` or a simple `for` loop etc. What I care about is understanding **What problem is this code really trying to solve?** Code should be written in such a way that I can just read what it's doing as-if I were reading a book. At least, that's what I prefer :) All that to say, the first benefit to using fluent interfaces is that **it gives the developer a platform or technique for building code which is being designed with the intent that other people will actually be reading (and maybe reusing) this code.** It's not a silver bullet, but it can be very powerful if approached with this in mind.

### It Forces You To Think About What Needs To Be Exposed

When dealing with C# based projects, I see the dreaded ["anemic objects"](https://en.wikipedia.org/wiki/Anemic_domain_model) all the time. Basically, these are objects which have a bunch of getters and setters. Instead of the object offering it's own behavior to others, it just lets everyone else mess around with all of it's data. When using fluent interfaces I find that it forces me to start with a 100% locked object, and begin to add methods that offer the behavior I need. **Instead of thinking about "what data does this object have", I find I am forced to think about "what kinds of thing can this object do?"**

### Helps Avoid Other Design Problems

Another design flaw I have seen in my day (yes.... in real production code) are methods with a bijillion parameters. I.e.

    public void SearchDatabaseForCustomers(string filterCountry, string filterUsername, bool onlyDelinquents, int selectLimit .... )
    

Ya, it's gross. Moving on, however.... I find that using fluent interfaces make me think in steps of behavior. As opposed to thinking about doing a whole bunch of stuff at once, I know that I'm trying to build expressive and meaningful method names. Because of method chaining, the value given to one method can affect the effect of the next method in the chain. Using the validator theme from my [previous article](https://www.blog.jamesmichaelhickey.com/exploring-fluent-interface/):

    validator.When("user.address.province")
             .IsNot("ON")
             .ThenFail();
    

It would be easy to build one method that accepts all the inputs as parameters. But eventually problems will arise. By using fluent interfaces, all new functionality can just be added via a new method.

    validator.When("user.address.province")
             .IsNot("ON")
             .ThenFail()
             .WhenFailed(() => ... do some stuff)
             .WhenPassed(() => ... do some stuff)
    

This code above, if using one method to do it all, would need to add two more parameters to supply the callback functionality.

If You Enjoyed This...
======================

Want to see an example of how to use functional programming to improve your C# code? [Functional Programming With C#: A Simple Use Case](https://www.blog.jamesmichaelhickey.com/csharp-functional-programming-a-simple-use-case/) **Let me know what you think! Thanks for reading!**