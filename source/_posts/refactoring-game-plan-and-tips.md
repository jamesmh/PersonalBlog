---
title: "Refactoring Legacy Monoliths - Part 3: Game Plan And Refactoring Tips"
tags:
  - legacy code
  - legacy software
  - monolith
  - refactoring
url: 581.html
id: 581
categories:
  - "C#"
  - Object Oriented Programming
  - Refactoring
  - Software Architecture And Design
date: 2018-03-07 16:25:04
---

So your engineering team is convinced that you need to make some drastic changes. The direction of future development needs to improve. Things can't stay as they are. Management is also convinced that the product needs to move in a new direction. What's next? Well, before doing any actual changes or refactoring to your product, planning a refactor is your next step. In other words, you need a game plan. I'll also discuss some refactoring tips for you to get started!

<!--more-->

# A Word From Our Sponsors...

P.s. This is part 3 of my "Refactoring Legacy Monoliths" series:

[Refactoring Legacy Monoliths - Part 1: First Steps](https://www.blog.jamesmichaelhickey.com/refactoring-legacy-monoliths-first-steps/)
[Refactoring Legacy Monoliths - Part 2: Cost-Benefit Analysis Of Refactoring](https://www.blog.jamesmichaelhickey.com/refactoring-legacy-monoliths-part-2-convincing-management/)

# Does Refactoring Mean a Rewrite?

One comment I've seen come up on Reddit about this series (quite a bit...) is the accusation that I'm suggesting you ought to rewrite your entire codebase. I've never said that. Nor have I ever implied that you should. Really, you probably shouldn't.

# Trust

As a refresher, in his book Working Effectively With Legacy Code, Michael states:

> To me, legacy code is simple code without tests.

Why does Michael care so much about testing your code from the inside? (i.e. not by having people test your **website** over and over - which is really expensive btw). There's a simple question that can answer this:

> If you were to change a core feature of your product in a non-trivial way, would you feel confident about making that change? Would you trust the system to still act exactly as it did?

If you don't have any testing, then how can you be confident? How can you **trust** your system?

Having tests in place is like doing acrobatics in the air with a safety net vs. not having a safety net. Ouch!

# What Are Your Goals?

All that to say your first goal should be to start implementing unit tests on your code. This is foundational work. You need to be able to change your code and have confidence that it still works.

Again:

> Your first goal should be to implement code-based testing. You cannot refactor your system with confidence unless you have a safety net.

After this, your goals may vary. If you have completed [Step 1](https://www.blog.jamesmichaelhickey.com/refactoring-legacy-monoliths-first-steps/) and [Step 2](https://www.blog.jamesmichaelhickey.com/refactoring-legacy-monoliths-part-2-convincing-management/) then you should have a solid list of what needs to change.

What I would suggest at this point is having a formal discussion (with a formal outcome/document) that answers the following question:

> What do we want our system to look like in 1 year? In 2 years?

Maybe we should be using a new technology stack like ASP .NET Core? Maybe our current code architecture does not allow us to re-use our business rules in other platforms (web vs. mobile API vs. desktop app) - so we need to consolidate our business logic and rules. (P.s. None of these cases require a re-write)

# Dealing With Dependencies

The number one obstacle that (most likely) prevents you from creating isolated unit tests and isolating your business rules and entities are **dependencies**.

Once you start, you find that you start telling yourself:

> Well, in other to test [thing 1] I now need to have an instance of [thing 2]. But, [thing 2] needs an instance of [thing 3].

"Thing 1" might be an entity you want to test - let's say, a `Report` entity (which models some tabular data).

Now, imagine that "Thing 2" is another class - `LinkGenerator` (which generates links for the report).

`LinkGenerator` needs access to "Thing 3", which is, the `HttpSession`.

If you want to unit test the `Report` entity, you need:

- an instance of `LinkGenerator` which needs...
- an instance of `HttpSession`

Uh Oh. How can you unit test when you need `HttpSession`? Unit tests don't run off a web server! (Well, they shouldn't...)

Sorry to say (you already know...), it's going to take some work. You need to **break the chain of dependencies.**

Fortunately for us, that's one of the primary goals of refactoring. Others have already done the hard lifting for us.

# Dependency Breaking And Refactoring Tips

Let's look at a couple dependency breaking refactoring tips.

### 1. Turn Globals Into Interfaces. Then Inject Them.

The title says it all. Sticking with our little example, imagine the `LinkGenerator` has the following method (pseudo-ish code).

```
public string GenerateLink()
{
     // ... some kind of processing
     var someValue = HttpSession["SomeKey"];
     // ... more processing
     var someOtherValue = HttpSession["SomeOtherKey"];
     return link;
}
```

We can't test this method because it references the `HttpSession` object that only exists in a web application. We don't want our models or business entities to know about the web (this is in line with our goal of isolating business entities from the presentation of our data).

By injecting an interface instead, we can remove the dependency on the actual `HttpSession`.

```
public string GenerateLink(IHttpSessionAccessor session)
{
     // ... some kind of processing
     var someValue = session.GetValue("SomeKey");
     // ... more processing
     var someOtherValue = session.GetValue("SomeOtherKey");
     return link;
}
```

I'm sure you can imagine what the interface definition would look like. The concrete class might look something like this:

```
public class HttpSessionAccessor : IHttpSessionAccessor
{
    private readonly HttpSession _session;

    public HttpSessionAccessor(HttpSession session)
    {
        this._session = session;
    }

    // You could be fancy and use generics?
    public object GetValue(string key)
    {
        return this._session[key];
    }
}
```

Now, we can do something like this in our testing code:

```
IHttpSessionAccessor session = new Mock<IHttpSessionAccessor>();

// Implement the mock
// Or just assign "session" with a dummy implementation of IHttpSessionAccessor.

LinkGenerator generator = new LinkGenerator();
string link = generator.GenerateLink(session);

// Assert ...
```

### 2. Adapt Parameter

Imagine our code above was originally this:

```
public string GenerateLink(HttpSession session){
     // ... some kind of processing
     var someValue = session["SomeKey"];
     // ... more processing
     var someOtherValue = session["SomeOtherKey"];
     return link;
}
```

What's wrong? Well, we have the same issue as above. We **still** need an actual instance of `HttpSession`. We need to run the tests on an actual Http request. So, we need a web server to be running...

To solve this, just do the same thing as #1. Turn the parameter into an interface and access the interface instead of the actual implementation (HttpSession).

### 3. Extract Method

You are probably familiar with this technique. If you have a section of code that is doing multiple chunks or has multiple responsibilities, then you need to break them up. Take a chunk of code that does one thing, and create a new method out of it. Avoid references to global state (like `HttpSession`) so that you can unit test your new methods.

Good indicators of where to break up your code are:

- Comments saying "Do this", then "Do this next thing" inside the same code block are usually each a candidate for extraction.
- C# regions are a **really good** indicator that you need to split this code up!

### Conclusion

The primary areas you need to focus on are:

- Building a game plan describing **where you want your system to be in 1 year**
- Making your software **trustable**
- Being able to have **confidence** after changes are made to the code

Dependencies will need to be broken. But this ultimately leads you to a place where:

- Your code is testable
- Your overall design is (overall) better
- You can trust the system/software after changes

Thanks for reading :) Let me know what you think in the comments. Have you ever had to go through this process personally? Let me know!

# Keep In Touch

Don't forget to connect with me on [twitter](https://twitter.com/jamesmh_dev) or [LinkedIn](https://www.linkedin.com/in/jamesmhickey/)!

I also have an e-mail letter where I'll give you tips, stories and links to **help ambitious and passionate developers become tech leaders.** I'll also give you updates about stuff that I've been working on ;)

[Subscribe if you haven't already!](https://tinyletter.com/jamesmh)

# P.S.

I've been building a tool for indie .NET Core developers needing to get their next groundbreaking app or side-project to market faster - without compromising code quality and elegance. [It's called Coravel!](https://github.com/jamesmh/coravel). Check it out and let me know what you think ;)
