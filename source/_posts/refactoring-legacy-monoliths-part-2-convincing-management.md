---
title: 'Refactoring Legacy Monoliths - Part 2: Cost-Benefit Analysis Of Refactoring'
tags:
  - legacy monoliths
  - legacy software
  - maintainability
  - refactoring
  - software design
url: 566.html
id: 566
categories:
  - Refactoring
  - Software Architecture And Design
date: 2018-02-09 07:55:02
---

How do you convince management to invest time and money into refactoring your legacy monolith? Convincing management that benefits of refactoring are worthwhile can be a stopping point for many. It's your job to provide a cost-benefit analysis of refactoring your legacy monolith and convince your team that this makes sense. Btw, this is a sequel to my [previous post on this topic](https://www.blog.jamesmichaelhickey.com/refactoring-legacy-monoliths-first-steps/) where I discussed a starting point when facing this topic:

*   Code that can be tested (at all!)
*   And... have tests
*   Business logic separated from your presentational logic
*   Stop wasting time building code that is already available in stable/tested 3rd party libraries
*   etc.

Let's look at overcoming the next hurdle in this process.

Roadblock
=========

At this point, you and your dev team are all on board, but you (or your senior developer) doesn't have the last say on big decisions like this. So, you need to convince them that a large scale refactoring is needed. But there's a problem: **your company is not interested in _making software_, they want to _make money_ and _please their customers_**. This is how your non-developer manager or company owner (even if he is a developer :) ) will probably think this through:

> Spending weeks and months changing a bunch of code (i.e. refactoring) doesn't really affect our users! It doesn't even change what the customers see and how they use our product. Instead, we should build new features to give our customers more value. Afterall, more value given to our user = more money in our pocket.

So, you have a problem. You **know** that the software you have to hard to maintain, hard to extend, can't be tested, and is not portable. But, that means nothing to the people who need to actually decide if they will pay you to do the required work. How do you overcome this?

You Need To Convert Developer Talk Into Money Talk
==================================================

The answer is pretty straightforward - but it just takes a bit of work. You need to convert each of your "developer benefits" into how they will save the company money. In other words, a formal report and analysis of the benefits of refactoring for your company.

> Convert all technical benefits into $$$$

You must research each area yourself. Based on where your code and system is, the final report will differ from product to product. Here's a small sample of what your report may look like and include:

* * *

#### Cost-benefit Analysis Of Refactoring _insert system here_ By _you_

*   Catching bugs during design time (by using test-driven development) saves the company [over 100 times the cost vs. fixing after deployment.](https://www.researchgate.net/figure/IBM-System-Science-Institute-Relative-Cost-of-Fixing-Defects_fig1_255965523)
    
*   Catching bugs during development (not using strict test driven development but implementing code based testing) can save the company [15 times the cost vs. fixing after deployment](https://www.researchgate.net/figure/IBM-System-Science-Institute-Relative-Cost-of-Fixing-Defects_fig1_255965523)
    
*   Implementing code reviews can save the company [30 hours of fixing missed software defects and maintenance](http://www.ifsq.org/finding-ia-2.html)
    
*   If we separate our business logic from our presentational logic it means now we can _actually_ build the following _using all the features of the current system_:
    
    *   Public API
    *   Mobile API
    *   Mobile App
    *   Desktop App
    *   etc.
*   Having a refactored codebase means future features will be _much_ quicker to develop (less time wasted on trying to figure our legacy design and extending it)
    
*   Having an industry standardized architecture enables new developers to be "up-and-running" much quicker
    
*   Using modern industry standardized technologies means:
    
    *   Faster development in general
    *   Company is more appealing to potential hires
    *   Enhanced security - less prone to security vulnerabilities

* * *

Final Tips
==========

Gather your research into "right-to-the-point" bullets, highlighting _how this will save money_. Then, present it to whoever needs to hears it. Make sure you include numbers and exact figures of how much time and/or money each point will save the company. This is key. "Developer benefits" doesn't mean anything to people who want their business to be profitable. Money and time are what matter. And don't forget - include reports (like in the example).

A Final Rabbit Trail
====================

Every decision a lead/senior developer needs to make should be made this way. For example, making developers "happier" by introducing various benefits like free coffee, more vacation time, remote options, etc. can be presented as a money saver. How? It increases employee retention and appeal of the company to potential hires. And, you can list these benefits as a marketing gimmick on your company's website to show everyone how great your company is - which can be appealing to potential customers.

Thanks!
=======

I hope that this was helpful and gave you some useful tips! Thanks for reading :) P.s. Here are some other posts you may enjoy:

*   [Functional Programming In C# - A Simple Use Case](https://www.blog.jamesmichaelhickey.com/csharp-functional-programming-a-simple-use-case/)
*   [YouTube Video: First Class Functions - Beginning Functional Programming In C#](https://www.youtube.com/watch?v=L5bP4FgENJQ)
*   [Deck The Halls With Strategy Pattern Implementations In C#: Basic To Advanced](https://www.blog.jamesmichaelhickey.com/strategy-pattern-implementations/)