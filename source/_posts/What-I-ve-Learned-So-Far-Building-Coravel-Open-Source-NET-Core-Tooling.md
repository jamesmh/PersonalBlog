---
title: "What I've Learned So Far Building Coravel (Open Source .NET Core Tooling) - Part 1"
date: 2018-09-28 14:54:24
tags:
  - Coravel
  - .NET Core 
  - ASP .NET Core
  - .NET Core Sass
  - Open Source Development
  - .NET Core Indie Developer
---

One night, about 4 months ago, I had a couple hours of free-time. I was really enjoying the benefits of building apps with .NET Core. But I felt it was still missing all the "built-in" features that makes [Laravel](https://laravel.com/) (PHP Framework) such a breeze to work with. 

After all, I just want to build awesome apps and not tinker with the same old boilerplate stuff. 

**What if I just want to schedule _my code_ to run once a week?** I don't want to configure Windows Task Scheduler. Or Cron. I just want to tell my code to schedule itself. Shouldn't that be easy to do (like with Laravel)?

<!-- more -->

# Beginning Of The Story

So I set out to learn some .NET Core while building an easy to use scheduler - similar to Laravel's. Just to challenge myself and see if I could do it. Eventually, "version 1" was completed.

I don't have much free time, so I need to make sure what I do is important. So I decided to tweet what I had on Twitter to **maybe** get some feedback. Who knows... 

To my surprise, [David Fowler](https://twitter.com/davidfowl) (.NET Core Architect) responded and the tweet gained **way more** interactions than I ever thought it would!

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">What if you could do this in your .Net Core apps? <a href="https://t.co/dj7JgbVELC">https://t.co/dj7JgbVELC</a><a href="https://twitter.com/hashtag/dotnet?src=hash&amp;ref_src=twsrc%5Etfw">#dotnet</a> <a href="https://twitter.com/hashtag/coravel?src=hash&amp;ref_src=twsrc%5Etfw">#coravel</a> <a href="https://twitter.com/hashtag/dotnetcore?src=hash&amp;ref_src=twsrc%5Etfw">#dotnetcore</a> <a href="https://t.co/KNHZwJbrg0">pic.twitter.com/KNHZwJbrg0</a></p>&mdash; James Hickey (@jamesmh_dev) <a href="https://twitter.com/jamesmh_dev/status/1010238650825756672?ref_src=twsrc%5Etfw">June 22, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

That was on a Friday. In just **two days** the [Coravel repo](https://github.com/jamesmh/coravel) became the top #4 trending C# repo on all of GitHub! Wow! I was (and still am) super humbled and surprised!

![Coravel trending on GitHub](/img/coravel-4th-github.png)

# Building More Features

Obviously, people thought this was a useful tool. So I decided to keep going and build other features that I personally would like available:

- Easy to use queuing
- A terse way to cache stuff
- Mailer that is easy to configure, has an expressive syntax and comes with out-of-the-box responsive e-mail templates
- CLI to generate files, etc.

When I had the time, I did my best to build them. Today, there are even more features than I had anticipated.

Originally, the scheduler was just a fun challenge. But now people were **actually using it**. So I had to re-write it to be more flexible, more unit tests, thread safety, etc.

# What Did I Learn?

Throughout this journey, I've learned **tons** - and am still learning each day.

I wanted to start documenting some of these things - and give tips to those wanting to build their own .NET based open source projects.

## 1. NuGet Gotcha's

I had no experience building NuGet packages. I've never done it before.

When I had originally built the Mailer feature, it was a separate .NET project that was referenced by the "main" Coravel project. My thinking was that NuGet packages just swallowed any referenced projects and included them.

**Wrong.**

Nuget packages don't do that. And, it sounds like they never will. 

**What do you do?** There are two solutions to my knowledge:

- Include them into your **one** package
- Package other projects as **separate NuGet** packages and reference them

The second option does force you to focus on building small modular packages.

But one of the driving philosophies behind Coravel is that it should be as easy and possible to get up-and-running. Multiple packages, in my eyes, was straying from this.

I chose to go with option 1. I didn't want to have to maintain more than one package (and all the extra maintenance that comes with it).

## 2. Listen To The Community

But what happens when you make the wrong choice?

I had comments from people around the idea of not wanting to include the Mailer's dependencies (MailKit, etc,) when just wanting to use, for example, the scheduler. Fair enough. 

So I ran I poll to see what others thought:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Question for peeps: <br><br>Right now Coravel includes MailKit as a dependency for the Mailer. Should the Mailer be a separate nuget so that using the Scheduler doesn&#39;t include MailKit? Should all the features be split?<a href="https://twitter.com/hashtag/AspNetCore?src=hash&amp;ref_src=twsrc%5Etfw">#AspNetCore</a> <a href="https://twitter.com/hashtag/ASPNET?src=hash&amp;ref_src=twsrc%5Etfw">#ASPNET</a> <a href="https://twitter.com/hashtag/Dotnet?src=hash&amp;ref_src=twsrc%5Etfw">#Dotnet</a> <a href="https://twitter.com/hashtag/dotnetcore?src=hash&amp;ref_src=twsrc%5Etfw">#dotnetcore</a> <a href="https://twitter.com/hashtag/coravel?src=hash&amp;ref_src=twsrc%5Etfw">#coravel</a></p>&mdash; James Hickey (@jamesmh_dev) <a href="https://twitter.com/jamesmh_dev/status/1035487034679455744?ref_src=twsrc%5Etfw">August 31, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Since this project is for others to use - it's important to know how people generally are wanting to use it.

But, I'm not fond of having to maintain multiple packages. The technical complexities this would introduce (splitting all the features) is not wanted right now either. 

So, I decided to go with splitting the Mailer into its own NuGet package. 

## 3. It's OK To Go Back And Fix Things

So, I had made the wrong choice in bundling the Mailer into the main Coravel package.

Is that a sin? Nope. Did I beat myself up because of it? Nope.

As your project grows you **discover how people are using your project and adapt**. **Actively listen** to how people want to use your project.

For example, I've had people mention they don't like the fact that Coravel is targeting .NET Core and **not** .NET Standard. 

Originally, Coravel was designed **only for** .NET Core apps. One of the main principles was that Coravel needed to just "hook" into the native .NET Core tooling so that it was seamless and super easy to configure. Especially making things hook into the service provider, `IHostedService` interface, etc.

However, slimming of some dependencies (due to some issues posted by others on GitHub) **caused me to learn more** about how .NET Core works. Due to those changes, switching to .NET Standard might actually make sense now. So I'm looking at investigating this as we speak.

And that's OK. I may have gone the "wrong" route originally. But **I'm still learning**. 

> It's OK to learn from your mistakes. That's how we gain experience. Go back and make it "right".

## 4. Building A Project Takes Time - Be In It For The Long-Haul

It's been 4 months since I started Coravel. Have I been coding every night for 4 hours non-stop? Nope.

I've got 7 kids. We homeschool. I'm married. I've got a full-time job. I've got a life. 

Do I have much free time? Nope.

The little bit of free time that I do get has to be allocated to a handful of "important" projects, such as this blog. 

If I spend time working on this blog then it's time that I won't be able to work on other things - like Coravel.

It's a juggling act for me. Probably for you too.

Having a long-term vision is what I need to ensure that I don't get overwhelmed when I don't get around to doing X for a few weeks.

And it's what you'll need if you want to build a lasting, useful and robust open source project. 

## 5. Pull Requests Take Time

Talking about time-suckers...

Pull-requests take **a lot** of time. **Way more than I had ever expected**. 

I'm super excited that people are interested in contributing to something that I've built. 

But open source doesn't mean that anyone can change projects to be what they want. I have a vision and philosophy behind everything I create in Coravel. Others can't read my mind.

It's my job then to make sure all contributions adhere to that vision. This means (for me) addressing what may seem like super insignificant details - one-word changes to documentation, variable naming, etc.

But, to me, all the small details matter. It's what makes Coravel unique. It's what makes it easy to use.

So don't feel bad when you aren't just accepting pull-requests right-and-left. **It's still your project**. 

# Until Next Time

I hope you enjoyed the story behind Coravel so far and some things I've learned along the way. 

I'd like to write another post soon that highlights the technical things I've learned - and get into some code!

# Other Parts In This Series

- [Part 2: Fluent APIs Make Developers Love Using Your .NET Libraries](https://builtwithdot.net/blog/fluent-apis-make-developers-love-using-your-net-libraries)

# P.S

I've been building [Coravel Pro](https://www.pro.coravel.net/) which is a suite of professional admin backend tools to help you kickstart and manage your next ground-breaking .NET Core app! Check it out!

# Keep In Touch

Don't forget to connect with me on [twitter](https://twitter.com/jamesmh_dev) or [LinkedIn](https://www.linkedin.com/in/jamesmhickey/)!

I have an e-mail letter where I'll give you tips, stories and links to **help ambitious and passionate developers become tech leaders.** I'll also give you updates about stuff that I've been working on ;)

[Subscribe if you haven't already!](https://tinyletter.com/jamesmh)




