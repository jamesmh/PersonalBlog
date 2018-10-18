---
title: Why Is .NET Core So Special? (Why You Should Use .NET Core)
tags:
  - .NET Core
categories:
  - .Net Core
date: 2018-10-18 00:43:27
---


With all the buzz around .NET Core I figured that I should tackle some of the fundamental issues that make .NET Core a gamechanger.

.NET Core __really__ is the next generation of .NET development and I believe that it's time for .NET developers everywhere to migrate!

<!-- more -->

# What Is .NET Core

For those new to .NET Core, I'd like to quickly explain what .NET Core is.

- Imagine being able to run C# apps (natively) in Linux.

- Imagine being able to use "npm like" tools that scaffold new projects for you.

- What if you didn't need Visual Studio anymore? What if you could build entire apps using VS Code? 🤯

.NET Core is basically the .NET languages (C#, F#, etc.) re-written onto a completely new runtime/sdk.

This runtime is not Windows specific. That means it runs on Linux, Mac and Windows.

It offers developers modern tooling that can scaffold projects, build, run, test and deploy using incredibly easy-to-use CLI tools.

.NET Core isn't simply a huge framework (like .NET Framework) that you built on top of. It's included as NuGet packages that you can use to carefully craft only the little tiny pieces that you need - if you do need that level of flexibility.

Since .NET apps have much less of a footprint, they are perfect for scenarios where you need to build small, high-performing, isolated applications - Micro-services.

# Who Is Using .NET Core Anyways?

Aren't all the massive/large scale apps using Nodejs though? Nodejs is typically touted as the highest performing and most scalable platform for building modern web apps. Is that true?

I want to go through some of the companies and products that are currently using .NET Core in production (taken from [these case studies on the official .NET site](https://www.microsoft.com/net/platform/customers)). We'll look at why they decided to use .NET Core and what benefits they found were critical to their businesses.

## RayGun: High Performance

Quoting the official site:

> Using the same-size server, we were able to go from 1,000 requests per second per node with Node.js to 20,000 requests per second with .NET Core.

That's pretty amazing! That's an increase of __2000%__ in terms of requests per second. Imagine how much money you could save by moving to a smaller hosting environment because you don't need so much "juice"?

## Siemens Healthineers: Linux Enabled

>  It also gives us strong benefits with regard to operation costs in the cloud, because we can use it to run some workloads on Linux machines.

Not only does .NET Core allow you to run-more-code-on-less-machine, but you can also run an entire .NET Core app within a Linux operating system. Since they are generally free, the cost savings of __not__ requiring paid OS licenses can save __a lot__ of money.

## GoDaddy: Scalability

> Services can be developed more quickly, perform faster in production, and scale better if they’re written using .NET Core

> .NET Core gives us the freedom to take advantage of new infrastructure technologies that run on Linux such as Kubernetes and Docker.

.NET Core was designed to allow you to create small isolated services and scale them independently if needed. 

You don't need to buy a new massive server just because one small part of your app is seeing a higher load. Just build a .NET Core app and stick it into a container. 

Now you can infinitely scale your app as needed!

## VQ Communications: Freedom and Flexibility

> The fact that .NET Core is cross-platform allows developers more freedom in how they develop the product because, at the end of the day, it's going to run on .NET Core, and that will be macOS, Linux, or Windows

.NET Core can target Linux, Windows or Mac. This means you can build .NET Core apps using a Mac or Ubuntu - using VS Code or Sublime Text. You have more freedom now to use the tools that work for you.

Personally, I've been using VS Code to build all my .NET Core apps and libraries. I don't need Windows. I don't need Visual Studio. And it's great!

## Age Of Ascent: Open For Contributions And Improvements

> ASP.NET is open source, that allows us to contribute back to it if we have any performance issues which Microsoft review and together we make a better product.

Ben Adams is considered one of the most knowledgeable .NET developers in the world. He doesn't work for Microsoft - he's the CTO of Illyriad Games.

Since .NET Core is open source - Ben has been able to be a part of many non-Microsoft employed developers who have made .NET Core more performant, added new features and provided insights into the product.

# What About Indie And Side Projects?

"But I'm not some super sized-organization" - you might say. What about building my side-projects quickly?

.NET Core gives you some fantastic tools that can accelerate your development:

- Built-in Dependency Injection
- Easy and isolated configuration (no more web.config!)
- Razor Pages (a new type of project that allows you to build web apps quickly)
- Easy database access with Entity Framework Core
- Fantastic CLI tools to scaffold and build your apps with more productivity

On top of all that, I've been building an open source library that can accelerate building .NET Core web apps even faster!

[Coravel](https://github.com/jamesmh/coravel) gives you a near-zero config set of tools such as Task Scheduling, Queuing, Caching and a CLI that lets you scaffold even more so you can be super productive.

# How To Get Started

Thinking about building your next project using .NET Core? [Start here.](https://docs.microsoft.com/en-us/aspnet/core/getting-started/?view=aspnetcore-2.1)

Do you have a .NET Framework app that you are considering migrating to .NET Core? [Start here.](https://docs.microsoft.com/en-us/aspnet/core/migration/?view=aspnetcore-2.1)

Already know how to use .NET Core but need more tools that will help you build fully featured web apps? [Start here.](https://github.com/jamesmh/coravel)

# Keep In Touch

Don't forget to connect with me on [twitter](https://twitter.com/jamesmh_dev) or [LinkedIn](https://www.linkedin.com/in/jamesmhickey/)!

# P.S

I've been building [Coravel Pro](https://mailchi.mp/2ab47aaa76c9/coravelpro) which is a suite of professional admin tools to help you kickstart and manage your next ground-breaking .NET Core app! Check out the [landing page](https://mailchi.mp/2ab47aaa76c9/coravelpro) to see the features and keep up-to-date with the project.