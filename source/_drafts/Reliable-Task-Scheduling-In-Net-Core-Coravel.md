---
title: 'Reliable Task Scheduling In .NET Core With Coravel'
tags: 
    - .net core task scheduling
    - task scheduling c#
    - task scheduling
    - windows services
---

This post is part of the the [2018 C# Advent Calendar](https://crosscuttingconcerns.com/The-Second-Annual-C-Advent).

In the spirit of the season, we'll be discussing how Santa Clause has recently been using .NET Core to build his internal Christmas present processing system.

# Our Context

Santa is an intermediate developer but is learning the ins-and-outs of .NET Core. His system needed to be robust in terms of security but also with a focus on ease of development. He's decided that .NET Core is the best choice when considering these criteria.

He has **very** tight deadlines though (as you can imagine).

Santa didn't want to re-invent the wheel - but he **needed** a reliable yet simple way to schedule background tasks, queue work so his web app was responsive (mostly for the elves), etc.

# Coravel

He came across [Coravel](https://github.com/jamesmh/coravel/blob/master/README.md) - which is a near-zero config open source library for .NET Core developers. It focuses on helping developers get their web applications up-and-running fast - without compromising code quality.

Because it's written specifically as a set of tools for .NET Core, it takes advantage of native features like the built-in dependency injection services, `IHostedService`, etc.

Santa has really enjoyed using Coravel - especially time savings due to not having to configure and install other heavy dependencies for scheduling, queuing, event broadcasting, etc. individually.

# Drawback

Santa now has to schedule some **really** long running tasks. Tasks that might take hours to run. 

Doing this **inside** his ASP .NET Core application is not an option, since doing these types of long running tasks in a web app causes many issues.

# The Solution

Santa decided to check out [Coravel's GitHub repo](https://github.com/jamesmh/coravel/blob/master/README.md) - just in case this has been addressed before. 

It turns out that [there is a sample to address this exact concern!](https://github.com/jamesmh/coravel/blob/master/Samples/HostBuilderConsole/Program.cs) 

With Santa's permission, I asked if I could briefly share how he decided to implement this.

# Scheduling Tasks From A .NET Core Console Application

One of the benefits of Coravel being a .NET Core native library is that it's **so simple to configure**. 

Combined with one of .NET Core's best features, `HostBuilder`, and you can do some really powerful things in just a few lines of code.

The `HostBuilder`, by the way, let's you construct a .NET Core application by adding just the specific pieces you need. Then you can "host" whatever you need (mini-API endpoints or multiple hosted services) without the full dependencies needed for a typical web API project etc.

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

This will be a console app that hooks into Coravel's scheduler. Every minute something will happen.

# Maintainable Backend Jobs With DI Support

Santa needed a clean and maintainable way to manage his backend jobs. 

He has dozens of these backend jobs. 

For example...

Was there also a way to gain access to his Entity Framework Core context in order to persist and query his database?

Luckily for him, Invocables address all of these pain points.

What if we introduced one of my favorite features of Coravel - [Invocable classes](https://github.com/jamesmh/coravel/blob/master/Docs/Invocables.md)?

> An Invocable is a class that represents some "job" in your application:
> - Sending automated emails
> - Cleaning your database
> - Processing messages from an external queue
> - Syncing data between an external API and your system





