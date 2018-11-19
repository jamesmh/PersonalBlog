---
title: 'Simple And Scalable Task Scheduling In .NET Core With Coravel'
tags: 
    - .net core task scheduling
    - task scheduling c#
    - task scheduling
    - windows services
---

This post is part of the [2018 C# Advent Calendar](https://crosscuttingconcerns.com/The-Second-Annual-C-Advent).

In the spirit of the season, we'll be discussing how Santa Clause has recently been using .NET Core to build his internal Christmas present processing system.

<!-- more -->

# Santa's Needs

Santa is an intermediate developer but is learning the ins-and-outs of .NET Core. His system needed to be robust in terms of security but also with a focus on ease of development. 

He had decided that .NET Core was the best choice when considering these criteria.

Santa didn't want to re-invent the wheel - but he needed a reliable yet simple way to schedule background tasks, queue work so his web app was responsive (mostly for the elves), etc.

# Coravel

He came across [Coravel](https://github.com/jamesmh/coravel/blob/master/README.md) - which is a near-zero config open source library for .NET Core developers. 

Coravel focuses on helping developers get their web applications up-and-running fast - without compromising code quality. Features like task scheduling, queuing, caching, event broadcasting, etc. are available right out-of-the-box.

Because it's written specifically as a set of tools for .NET Core, it takes advantage of native features - such as full support for the built-in dependency injection services. 

For example, you can inject dependencies/services into scheduled tasks, queued tasks, event listeners, etc. with zero fuss and no extra configuration!

Santa has really enjoyed using Coravel - especially the time savings gained from not having to configure and install other dependencies for scheduling, queuing, event broadcasting, etc. individually. 

He also enjoys the terse and simple syntax that Coravel offers.

# Drawback

But - now Santa has to schedule some **really** long running tasks. Tasks that might take hours to run. And these are important tasks.

Doing this **inside** his ASP .NET Core application is not an option since doing these types of long-running tasks in a web app causes many issues (as you probably know).

# The Solution

Santa decided to check out [Coravel's GitHub repo](https://github.com/jamesmh/coravel/blob/master/README.md) - just in case this has been addressed before. 

It turns out that [there is a sample to address this exact concern!](https://github.com/jamesmh/coravel/blob/master/Samples/HostBuilderConsole/Program.cs) 

I Santa asked if I could briefly share how he decided to implement this. He agreed, but I was only permitted to show a very small sample of his system.

# Scheduling Tasks From .NET Core Console Applications

One of the benefits of Coravel being a .NET Core native library is that it's **so simple to configure**. 

Combined with one of .NET Core's coolest features, `HostBuilder`, and you can do some really powerful things in just a few lines of code.

The `HostBuilder`, by the way, let's you construct a .NET Core application by adding just the specific pieces you need. Then you can "host" whatever you need (mini-API endpoints or multiple hosted services) without the full dependencies needed for a typical full web project.

Using the sample mentioned above, let's look at a very basic implementation of using `HostBuilder` along with Coravel's scheduling:

```c#
class Program
{
    static void Main(string[] args)
    {
        var host = new HostBuilder()
            .ConfigureAppConfiguration((hostContext, configApp) =>
            {
                configApp.SetBasePath(Directory.GetCurrentDirectory());
                configApp.AddEnvironmentVariables(prefix: "PREFIX_");
                configApp.AddCommandLine(args);
            })
            .ConfigureServices((hostContext, services) =>
            {
                // Add Coravel's Scheduling...
                services.AddScheduler();
            })
            .Build();

        // Configure the scheduled tasks....
        host.Services.UseScheduler(scheduler =>
            scheduler
                .Schedule(() => Console.WriteLine("This was scheduled every minute."))
                .EveryMinute()
        );

        // Run it!
        host.Run();
    }
}
```

This will be a console app that hooks into Coravel's scheduler. 

Every minute something will happen.

# Supporting .NET Core Dependency Injection With Invocables

Ok - that's a simple sample. Santa needs something more maintainable and he **really** needs to inject his Entity Framework Core Db context, `HttpClient`, etc. into his scheduled tasks.

With Coravel you can use [Invocable](https://github.com/jamesmh/coravel/blob/master/Docs/Invocables.md) classes to solve this problem.

> Invocables are ubiquitous classes that can be scheduled, queued, etc. with full support for .NET Core dependency injection. 
> 
> They represent some "job" in your application:
> - Sending automated emails
> - Cleaning your database
> - Processing messages from an external queue
> - Syncing data between an external API and your system

# Santa's Invocables

Santa has an API that he uses to fetch who is nice and naughty (it's a secret end-point from his legacy system). He then needs to do some CPU intensive processing to validate the data, and then store the results in his database. 

He needs this done once every hour using the invocable classes he's created. 

```c#
scheduler
    .Schedule<PutNaughtyChildrenFromAPIIntoDb>()
    .Hourly();

scheduler
    .Schedule<PutNiceChildrenFromAPIIntoDb>()
    .Hourly();    
```

# Problem With Scalability

Coravel internally uses `Task.WhenAll` to make sure async calls within scheduled tasks are processed efficiently. However, that does mean CPU intensive tasks could "hog" the thread that is processing the currently due tasks - causing other tasks to "pause".

This design makes sure that in web application scenarios Coravel won't be hogging threads that could otherwise be (and should be) used to respond to HTTP requests.

But Santa is specifically using a console application so he doesn't need to worry about that! What should he do?

# Schedule Workers

Coravel solves this problem with [Schedule Workers](https://github.com/jamesmh/coravel/blob/master/Docs/Scheduler.md#schedule-workers).

By using schedule workers Santa can put each of these tasks onto their own dedicated pipeline/thread(s):

```c#
scheduler.OnWorker("NaughtyWorker");
scheduler
    .Schedule<PutNaughtyChildrenFromAPIIntoDb>()
    .Hourly();

scheduler.OnWorker("NiceWorker");
scheduler
    .Schedule<PutNiceChildrenFromAPIIntoDb>()
    .Hourly();    
```

This will now execute each task in parallel so Santa can fetch and process both of these more efficiently. Any intensive CPU work that either task may perform will not cause the other to wait.

# Schedule Task Groups

In some cases, you may want to put multiple tasks onto one worker and perhaps dedicate one worker for a task that is known to either take a long time or is just CPU intensive.

For example:

```c#
scheduler.OnWorker("EmailTasks");
scheduler
    .Schedule<SendNightlyReportsEmailJob>().Daily();
scheduler
    .Schedule<SendPendingNotifications>().EveryMinute();

scheduler.OnWorker("CPUIntensiveTasks");
scheduler
    .Schedule<RebuildStaticCachedData>().Hourly();
```

The first two tasks don't take that long to complete and are not a priority in terms of CPU usage.

The second, however, needs to be able to work without being blocked.

This setup will ensure that `RebuildStaticCachedData` is always executed in isolation with it's own dedicated pipeline.

# Conclusion

I hope you've enjoyed this article and would love to hear from you in the comments! Maybe there's a better way to do this? Maybe you prefer some other way? I'd love to hear from you!

# P.S.

I've been building [Coravel Pro](https://www.pro.coravel.net/) which is a suite of professional admin backend tools to help you kickstart and manage your next ground-breaking .NET Core app! Check it out!

## You Might Enjoy

- [What Makes .NET Core So Special?](https://www.blog.jamesmichaelhickey.com/What-Makes-NET-Core-So-Special-Why-You-Should-Use-NET-Core/)
- [What I've Learned So Far Building Coravel (Open Source .NET Core Tooling)](https://www.blog.jamesmichaelhickey.com/What-I-ve-Learned-So-Far-Building-Coravel-Open-Source-NET-Core-Tooling/)
- [Fluent APIs Make Developers Love Using Your .NET Libraries](https://builtwithdot.net/blog/fluent-apis-make-developers-love-using-your-net-libraries)

## Keep In Touch

Don't forget to connect with me on [twitter](https://twitter.com/jamesmh_dev) or [LinkedIn](https://www.linkedin.com/in/jamesmhickey/)!

I also have an e-mail letter where I'll give you tips, stories and links to **help ambitious and passionate developers become tech leaders.** I'll also give you updates about stuff that I've been working on ;)

[Subscribe if you haven't already!](https://tinyletter.com/jamesmh)


