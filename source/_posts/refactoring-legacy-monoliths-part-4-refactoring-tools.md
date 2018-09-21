---
title: "Refactoring Legacy Monoliths – Part 4: Refactoring Tools"
tags:
  - "c#"
  - legacy software
  - maintainability
  - refactoring
  - software design
  - tools
url: 627.html
id: 627
categories:
  - Refactoring
  - Software Architecture And Design
date: 2018-05-29 16:47:59
---

As a software developer it's important to know what tools are available to you. Tedious and repetitive tasks or large "one-off" time-consuming tasks can often be automated by third-party tooling. And yes - sometimes it's even worth purchasing some of these tools with your own money. Specifically, when refactoring, we should have some knowledge of what refactoring tools are available to us.

Continuing my "Refactoring Legacy Monoliths" series - I want to go over a few tools that I've found super helpful and worth investing in.

To make this blog post more useful than a list of products, I'll go through some high-level steps that represent a way to tackle a refactoring project.

<!--more-->

# Step 1: Identification Of Problem Areas

Getting an overview of where pain-points are in your code-base has to be done. Why? Well, how do you know **for sure** that some code **ought** to be refactored? Because you _feel like it_? Because of _ugly code_?

Unless you measure your code-base in the same way that you would measure - let's say - the performance of a real application, you don't **really** know where issues really are. You might have _educated guesses_ about what needs to change. But not objectively quantifiable conclusions.

The best tool I've found to get this kind of objective view of your code-base is [NDepend](https://www.ndepend.com/).

NDepend integrates right into Visual Studio and basically just adds it's own menu etc. It can be run via an external .exe if you want to.

NDepend has so many features it's almost overwhelming (which is a good thing). I'll mention a couple features that I've found the most helpful in getting a high-level overview of our code-base.

## NDepend Feature: Code Treemap (or "Heat Map")

This feature is called a [Treemap diagram](https://www.ndepend.com/features/code-complexity#Diagrams) but reminds me of a Heat Map for your code. It colours specific areas of the diagram (modules in your code) according to their code quality (red is bad...). It reminds me of hard disk usage diagrams that show you what applications in your system are using lots of space, etc.

This is a super quick and reliable way to see where your investigation might want to start. Large red areas are immediate candidates for further investigation. And, most likely, represent some critical part of your app. Again, I said _most likely_.

In my professional experience, the large red areas almost always correspond to where most "bugs" in the software are eventually found.

If you are a consultant who deals with code-bases in any form, but more specifically in offering code-refactoring or code-analysis services, then I would say this feature is maybe worth the entire cost of the tool. It's just so easy to get started - especially with huge code-bases.

## NDepend Feature: Dashboard

Just like any great piece of software, NDepend has an awesome dashboard.

![NDepend dashboard](https://www.ndepend.com/Doc/VS_CQL/Dashboard.png)

From here you can drill into "issues" which gives you a list, ordered by severity, of specific issues in your code-base. Taking note of the most common wide-spread issues give you an idea of the overall technical concerns you might have about the code-base.

Right away, in your face, is a "grade" of how much technical debt the overall code-base has accumulated. These metrics are customizable too, in the event you have certain measurements you feel are not needed to analyze.

This is a time-saver and gives you super-powers for gaining insight into how a code-base was built and what specific issues need to be dealt with. If you are new to a project, you can - within 15 minutes - have the ability to say to those experienced in that project:

> "Hey [team-mate], I've had a quick look at the code to get a high-level overview. Let me take a guess and say that [module name] has probably been causing issues for you? Lots of bugs? Most of the team doesn't like to work in that area?"

This can be a way of having others trust you quicker than they would have otherwise.

## Roslyn Analysers In C

Roslyn analyzers are native C# code analyzers that you can use in Visual Studio. Visual Studio 2017 has the ability to run a code analysis on your entire solution and also has a feature that will calculate code metrics - similar to NDepend.

This can be an easy way to analyze your code-base if you are already using Visual Studio. The cool thing is that you can install different Roslyn analyzers into Visual Studio to enhance your intellisense, real-time suggestions, build time error checking, etc.

Running these metrics/analyzers will show up in your code (as a squiggly) and in the error list in Visual Studio. You can export the code metrics to an excel file, which is nice.

These features aren't comparable to the power of NDepend, but I think these represent an area that Visual Studio has been lacking in. I'm excited to see if Microsoft will add some advanced reporting on the analyzers like charts, dashboards, etc.

# Step 2: Gain Momentum By Tackling Safer Areas First

Once you know where the problem areas are, I would suggest tackling the smaller red areas (from the Treemap). These represent areas that don't have an overwhelming amount of code, yet do need to be refactored. These areas are probably known as "small" bug producers.

Think of the system you are working with as a **"bug producing onion"**.

> In the center is the largest, most horrible, horrendous piece of the system known to frighten even the mightiest at heart. As we move closer to the outside of the onion we encounter error-prone but less intimidating and less crucial pieces of the system.

Let's start by unravelling the onion from the outside.

This approach is less risky, allows you to build up confidence and domain knowledge (if working in a system you're not so familiar with).

But **how** do you begin tackling these areas?

## NDepend

Using NDepend's dashboard, you can see a list of specific code quality issues. Drilling into each issue will give you tips on how to fix those specific issues. I encourage you to explore this feature yourself :)

## ReSharper

Moving onto another tool though, [JetBrain's ReSharper](https://www.jetbrains.com/resharper/) is fantastic. Just like NDepend, ReSharper has tons of features. It integrates into Visual Studio seamlessly. It is known to be resource intensive on larger project/solutions, but for the sake of refactoring code, it's invaluable.

### ReSharper Feature: Move Types into Matching Files

This was an issue I had specifically on a project I was refactoring. There was one file which had **hundreds** of different types (classes, interfaces, etc.) that were in **one** file named "WIP.cs". That's short for "Work In Progress". And yes, this was in production code. Some types were actively used. Some weren't. And, some had version numbers appended to them - like "Class2" and "Class3". Fun.

So, instead of going through each of the types one-by-one and extracting them into their own files, ReSharper has a command that you can run on this file that will just magically do that for you. Just that saved me days of work - not to mention my sanity.

### ReSharper Feature: Adjust (Solution) Namespaces

I now had a few hundred extra files to deal with - all in the same folder. Next, I needed to structure the files as best as I could with a proper file/folder structure. This was manual work - but once all the files are in their proper place we still need to adjust all the namespaces for each type!

ReSharper has that covered. Just run the "Adjust Namespaces" command on a specific project, file or the entire solution and the folder structure for each file will be automatically applied to the namespace of the file. Again, that saved me _at least_ a few days of work.

This feature shines when doing high-level re-organization refactoring (i.e. file structure and folder structure changes).

From this real-life example, I was able to perform what otherwise would have taken weeks of work and get it done in a few days. Win!

# Appendix: Performance And Web Metrics

After you refactor your code and have some unit tests in place (yes, you really should...), I always like to profile run-time performance of the code that was changed. This next tool is one I **always** have running **all the time**. As a web developer (primarily) this tool is non-negotiable.

## Stackify Prefix

[Prefix](https://stackify.com/prefix/) is a **free** real-time web profiler. Basically, you install it on your machine and then hit a local URL that Prefix is assigned to. Voila!

It will automatically show you every HTTP request that your machine is handling with **tons** of details about each request. It's not a generic performance profiler - Prefix will show you **specific details** about each request. Features I've come to love are:

- It will show you hidden exceptions you never knew were happening (and are slowing your app down)

- All database calls are displayed with the actual code that was executed, how long the query took and even how long just downloading the data from the SQL server took (helpful with large data sets!)

- A code "hot-path" shows you which methods in your code are taking up the most time to execute (with the actual metrics displayed)

# Conclusion

To recap, NDepend, ReSharper, and Stackify's Prefix are all fantastic tools that boost your refactoring capabilities and ability to comprehend code from a high-level. They also give us tooling that will assist in the nitty-gritty details of improving our code.

Visual Studio code analysis tools offer an area of great potential. Roslyn analyzers, in particular, are an area where I see future potential integration into charts, dashboarding, etc. within Visual Studio to be a useful addition.

P.s. Here are some other posts you may enjoy:

- [How I Made LINQ 6X Faster Using A Functional Optimization!](https://www.blog.jamesmichaelhickey.com/linq-6x-faster-using-functional-optimization/)
- [Refactoring Legacy Monoliths Part 3: Refactoring Game Plan And Tips](https://www.blog.jamesmichaelhickey.com/refactoring-game-plan-and-tips/)
- [Refactoring Legacy Monoliths - Part 2: Cost-Benefit Analysis Of Refactoring](https://www.blog.jamesmichaelhickey.com/refactoring-legacy-monoliths-part-2-convincing-management/)
- [Deck The Halls With Strategy Pattern Implementations In C#: Basic To Advanced](https://www.blog.jamesmichaelhickey.com/strategy-pattern-implementations/)

# Keep In Touch

Don't forget to connect with me on [twitter](https://twitter.com/jamesmh_dev) or [LinkedIn](https://www.linkedin.com/in/jamesmhickey/)!

I also have an e-mail letter where I'll give you tips, stories and links to **help ambitious and passionate developers become tech leaders.** I'll also give you updates about stuff that I've been working on ;)

[Subscribe if you haven't already!](https://tinyletter.com/jamesmh)

# P.S.

I've been building a tool for indie .NET Core developers needing to get their next groundbreaking app or side-project to market faster - without compromising code quality and elegance. [It's called Coravel!](https://github.com/jamesmh/coravel). Check it out and let me know what you think ;)
