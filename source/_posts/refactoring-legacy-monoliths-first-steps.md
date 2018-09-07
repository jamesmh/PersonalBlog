---
title: "Refactoring Legacy Monoliths - Part 1: First Steps"
tags:
  - maintainability
  - object oriented design
  - OOP
  - refactoring
  - software design
url: 537.html
id: 537
categories:
  - Object Oriented Programming
  - Refactoring
  - Software Architecture And Design
date: 2018-01-09 20:20:05
---

Where do you even begin when considering "fixing" or refactoring legacy monoliths? I've been thinking about this lately - as I've been doing it for the last month or so.

<!--more-->

# What Is A Legacy Monolith?

In the book [Working Effectively with Legacy Code](https://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052), "legacy" software is merely defined as "code without tests." A few notes are made which include the idea that software with poor design or unmaintainable code is a large piece of what "legacy" means.

> "Refactoring legacy software" means refactoring your system to allow for code testing, and then implementing those tests.

A "monolith" is basically "all your code is one place." In other words, you have not separated different logical areas of your **product** into isolated areas.

If you are working on a .NET based web application (for example) and all your code is in one project (the web project) - then you most definitely have a monolith.

If your deploy process requires that the entire system is "pushed" when any new features are added or bugs are fixed - then that's a monolith.

# Realistic Goals

Before we being changing existing code, the idea that "legacy" code exists is an indication that the software process for this product is not working well. We need to consider a higher level question: "How do we even create software?"

Problems that you may experience in legacy projects, I think, boil down to one core issue:

> Software is not designed using known current industry standards.

What are some of these standards?

- Testing code
- Separating business logic from presentation logic
- Not re-inventing the wheel when reliable libraries exist to do the thing you need
- Avoiding depreciated third-party software (whether deemed so by vendor or development community)

Can you think of any more? I'd love to hear from you in the comments. :)

# Nip It In The Bud

So then, our first steps need to begin addressing these issues. But even before that, we need to realize that all future software creation is **still** following whatever process currently exists!

Having gone through this recently and creating a foundation for future development to - at the very least - **allow** for these standards to be followed, I think I have some insight. And so our first step is just that:

> Make the system at least **allow** future development to follow industry standards.

# Practical Steps

The most important point, which goes back to the definition of "legacy" software - is that you need code tests. Why?

- A clearer definition of system requirements (since tests correspond to individual business requirement)
- Forces your system to be modular (i.e. code that can be tested well is generally designed well)
- Early safety net when your code does something wrong

But, to be able to test properly, you need to have your business code (i.e. "business logic") separate from your (for example) web project. You can't test business rules unless you can test business logic on its own. Splitting your code into (at least) 2 different layers (business logic and presentation logic) is necessary for web projects (using it as an example) because:

> Your business code can rely on database access abstractions rather than database access implementations.

What? Let me ask you a question: "Can you test your code without having an active database available?" In true "legacy" projects, that's answered 99% of the time with "no".

What we need is a way to make our code rely on an `interface` which exposes `methods` that provide database access. Then, we can create our production code against that interface. Our system needs to somehow automatically "pass in" a concrete implementation of that `interface` to our business code.

Our tests, on the other hand, can use a stub or mock object which implement that `interface`. Our business code will still do what it does and not crash because there's no database connection.

# We're Thinking...

So we're thinking about the core issues and what we need to do to solve them. In the next part of the series, we'll look at trying to convince management that we do in fact need to change how software is made in our organization.

**Let me know what you think about the topic in the comments!** Thanks for reading!
