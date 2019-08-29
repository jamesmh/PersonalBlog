---
title: Why Should You Refactor Your Code?
date: 2019-08-29 08:43:10
tags: refactoring, code quality, clean code
---

**Refactoring** is just a fancy term that means improving your code without changing how it behaves. 

Let's look at some reasons for why you should learn about when and how to refactor effectively.
<!--more-->

_Note: This is an excerpt from my book [Refactoring TypeScript: Keeping Your Code Healthy.](https://leanpub.com/refactoringtypescript)_

[
![Refactoring TypeScript book](/img/refactoringts.png)
](https://leanpub.com/refactoringtypescript)

# Slow Development?

Ever work in a software system where your business asked you to build a new feature - but once you started digging into the existing code you discovered that it's not going to be so easy to implement?

Many times, this is because our existing code is not flexible enough to handle new behaviors the business wants to include in your application.

Why? 

_Well, sometimes we take shortcuts and hack stuff in._

Perhaps we don't have the knowledge and skills to know how to write healthy code?

Other times, timelines need to be met at the cost of introducing these shortcuts.

This is why refactoring is so important:

**Refactoring can help your currently restrictive code to become flexible and able/easy to extend once again.**

It's just like a garden. Sometimes you just need to get rid of the weeds!

> I've been in a software project where adding a checkbox to a screen was not possible given the system's set up at the time! Adding a button to a screen took days to figure out! And this was as a senior developer with a good number of years under my belt. Sadly, some systems are just very convoluted and hacked together.

> This is what happens when we don't keep our code healthy!

# Saving Money

It's a practical reality that you need to meet deadlines and get a functioning product out to customers. This could mean having to take shortcuts from time-to-time, depending on the context. Bringing value to your customers is what makes money for your company after-all.

Long-term, however, these "quick-fixes" or shortcuts lead to code that can be rigid, hard to understand, more prone to contain bugs, etc.

Improving and keeping code quality high leads to:

- Fewer bugs
- Ability to add new features faster
- Able to keep changes to existing code isolated
- Code that's easier to reason about

All of these benefits lead to less time spent on debugging, fixing problems, developers trying to understand how the code works, etc.

_E.g. It saves your company real money!_

## Aside: Navy SEALS Get It
There's an adage that comes from the Navy SEALs which many have noticed also applies to creating software:

> Slow is smooth. Smooth is fast.

Taking time to build quality code upfront will help your company move faster in the long-term. But even then, we don't anticipate all future changes to the code and still need to refactor from time-to-time.

# Being A Craftsman

Software development is a critical part of our society.

Developers build code that controls:

- Vehicles
- Power Grids
- Government Secrets
- Home Security
- Weapons
- Banking Accounts
- Etc.

I'm sure you can think of more cases where the software a developer creates is tied to the security and well-being of an individual or group of people.

Would you expect your doctor to haphazardly prescribe you medications without carefully ensuring that he/she knows what your medical condition is?

Wouldn't you want to have a vehicle mechanic who takes the time to ensure your vehicle's tires won't fall off while you are driving?

Being a craftsman is just another way to say that we should be professional and care about our craft. 

We should value quality software that will work as intended!

> I've had it happen before that my car's axel was replaced and, while I was driving away, the new axel fell right out of my car! Is that the kind of mechanic I can trust my business to? Nope!

> Likewise, the quality of software that we build can directly impact people's lives in real ways.

# Case Study #1

You might be familiar with an incident from 2018 where [a Boeing 737 crashed and killed all people on board](https://www.bloomberg.com/news/articles/2019-06-28/boeing-s-737-max-software-outsourced-to-9-an-hour-engineers). 

It was found that Boeing had outsourced its software development to developers who were not experienced in this particular industry: 

> Increasingly, the iconic American planemaker and its subcontractors have relied on temporary workers making as little as $9 an hour to develop and test software, often from countries lacking a deep background in aerospace.

Also, these developers were having to redo improperly written code over and over again:

> The coders from HCL were typically designing to specifications set by Boeing. Still, “it was controversial because it was far less efficient than Boeing engineers just writing the code,” Rabin said. Frequently, he recalled, “it took many rounds going back and forth because the code was not done correctly.”

One former software engineer with Boeing is quoted as saying:

> I was shocked that in a room full of a couple hundred mostly senior engineers we were being told that we weren’t needed.

While I have no beef with software developers from particular countries, it does concern me when a group of developers are lacking the knowledge or tools to build quality software in such critical systems.

For Boeing in general, what did this overall lack of quality and craftsmanship lead to?

**The company's stocks took a huge dip a couple of days after the crash.**

Oh, and don't forget - **people died.** No one can undo or fix this.

After it's all said and done, Boeing did not benefit from cutting costs, trying to rush their software development and focus on speed rather than quality.

As software development professionals, we should seek to do our part and value being software craftsmen and craftswomen who focus on creating quality software.

# Case Study #2

Do you still think that because airplanes can potentially kill people that the software built is going to be quality? Nope.

Here's another example of software quality issues in the aviation field: [$300 million Airbus software bug solved by "turning it off and on again every 149 hours."](https://gizmodo.com/turn-it-off-and-on-again-every-149-hours-is-a-concernin-1836818094)

Sound's kind of like a memory leak? You know, when you have a web application that starts getting slow and clunky after keeping it opened for a while. Just refresh the page and _voila_! Everything's fine again!

Sadly, we are building airplanes like that too...

Quoting the article:

> Airlines who haven’t performed a recent software update on certain models of the Airbus A350 are being told they must completely power cycle the aircraft every 149 hours or risk “...partial or total loss of some avionics systems or functions,” according to the EASA.

Do **you** want to fly on those planes?

Quality matters. And the fact is, many developers are **not** writing quality software. 

# How To Keep Your Code Healthy

This post was an excerpt from [Refactoring TypeScript](https://leanpub.com/refactoringtypescript) which is designed as an approachable and practical tool to help developers get better at building quality software. 

[
![Refactoring TypeScript book](/img/refactoringts.png)
](https://leanpub.com/refactoringtypescript)

## Keep In Touch

Don't forget to connect with me on [twitter](https://twitter.com/jamesmh_dev) or [LinkedIn](https://www.linkedin.com/in/jamesmhickey/)!

<div style="padding:0 20px; border-radius:6px; background-color: #efefef; margin-bottom:50px; margin-top:20px">
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

## You Might Also Enjoy

- [Do You Struggle Naming Your Classes Well?](https://www.blog.jamesmichaelhickey.com/Do-You-Struggle-Naming-Your-Classes-Well/)
- [Refactoring Legacy Monoliths - Part 1: First Steps](https://www.blog.jamesmichaelhickey.com/refactoring-legacy-monoliths-first-steps/)
- [Where Do I Put My Business Rules And Validation?](https://builtwithdot.net/blog/where-do-i-put-my-business-rules-and-validation)