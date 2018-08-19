---
title: 'Exploring Fluent Interfaces With C# (A.k.a. Fluent Interfaces Are Not Evil)'
tags:
  - domain driven
  - domain driven design
  - fluent
  - fluent api
  - fluent interface
url: 316.html
id: 316
categories:
  - 'C#'
  - Fluent Interfaces
  - Object Oriented Programming
  - Software Architecture And Design
date: 2017-09-01 01:33:07
---

Are fluent interfaces **evil**? (As some might suggest)... I don't think so. In fact, I think they are great. I plan on doing a few posts around this topic in the coming days / weeks (I'm pretty busy...). I wanted to start by addressing some common arguments I've come across. [First thing's first though...(if you don't know anything about fluent interfaces)](https://www.martinfowler.com/bliki/FluentInterface.html)

<!--more-->

### Encapsulation Broken

The first concern I've seen is that fluent interfaces somehow break encapsulation. Well...I disagree. If the only thing exposed by an object are methods "that do what you tell it to do", then it does stuff to it's private members - not the caller. Just because an object's method return's itself, you still have no access to it's private internals.

### Harder To Read

Again, I disagree. The problem with code that is not readable is that the developer has chosen a poor vocabulary. For example, here's some code that uses a fluent interface:

    validator.When("address.province")
             .Is("ON")
             .ThenFail();
    

Here's what you would normally see:

    bool fail = false;
    if(jsonObject["address"]["province"] == "ON")
    {
        fail = true;
    }
    

Which one just reads easier? I think the first. Now imagine complex / lengthy code...

### Too Many Choices

One last argument I see is that fluent interfaces leaves the user with too many methods to choose from. How does a user know which order to call methods in? My answer: there are ways to to limit what methods are available at any given moment in a method chain. There are two ways I know of to do this. (1) The fluent class implements various interfaces. When a method expects the next call in the chain to be restricted to a certain set of methods, it will simply return it's context (itself) casted to that interface. Going along with the validator theme:

    public IValidationResult Is(string someString) 
    {
       // Validation check of some kind
         return (IValidationResult) this; // IValidationResult defines ThenFail() and ThenPass().
    }
    
    // In caller...
    
    validator.When("province") // Now we are dealing with some interface that only exposes Is() and IsNot().
             .Is("ON") // Now we are dealing with an IValidationResult type, so our choices are restricted.
             .ThenFail(); // This could return the original type so we can start over using When().
    

(2) Have a collection of classes which pass the original context around to each other, with each class / type only exposing relevant methods.

    public FluentValidationResult Is(string someString)
    {
         // Validation check of some kind
         return new FluentValidationResult(this.validator); 
    
         // Above keeps a reference to the validator so we can 
         // re-expose it later in the method chain. We just keep passing
         // the validator onto the next class we expose.
    } 
    
    // In caller...
    
    validator.When("address.province") // Returns a new class that exposes the Is() and IsNot() methods.
             .Is("ON") // Returns a class that exposes the ThenFail() or ThenPass().
             .ThenFail(); // This would return the validator, allowing for further chaining.
    

### Next Time...

I don't think fluent interfaces are evil. Next time I think I would like to look at what I think are benefits to using fluent interfaces. **Do you agree? Disagree? Have I touched a nerve? Let me know in the comments!** Thanks for reading!

If You Enjoyed This...
======================

Want to see an example of how to use functional programming to improve your C# code? [Functional Programming With C#: A Simple Use Case](https://www.blog.jamesmichaelhickey.com/csharp-functional-programming-a-simple-use-case/)