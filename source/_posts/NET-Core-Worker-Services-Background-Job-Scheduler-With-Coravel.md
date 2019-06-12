---
title: '.NET Core Worker Services: Background Job Scheduler With Coravel'
tags:
  - .NET Core
  - Coravel
  - .NET Core Job Scheduler
date: 2019-06-12 16:06:23
---


The .NET Core CLI comes with tons of pre-built project templates! One of the new templates that will be included with [.NET Core 3](https://docs.microsoft.com/en-us/dotnet/core/whats-new/dotnet-core-3-0) will be for building worker services.

Combining .NET Core worker services with [Coravel](https://github.com/jamesmh/coravel) can help you build lightweight background job scheduling applications very quick. Let's take a look how you can do this in just a few minutes!

<!--more-->

_Note: Worker services are lightweight console applications that perform some type of background work like reading from a queue and processing work (like sending e-mails), performing some scheduled background jobs from our system, etc. These might be run as a daemon, windows service, etc._

# Installing .NET Core 3 Preview

At the writing on this article, .NET Core 3 is in preview. First, you must [install the SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0). You can use Visual Studio Code for everything else in this article 👍.

# Coravel's Task Scheduling

[Coravel](https://github.com/jamesmh/coravel) is a .NET Core library that gives you advanced application features out-of-the-box with near-zero config. [I was inspired by Laravel's ease of use](https://www.blog.jamesmichaelhickey.com/What-I-ve-Learned-So-Far-Building-Coravel-Open-Source-NET-Core-Tooling/) and wanted to bring that simple and accessible approach of building web applications to .NET Core.

One of those features is a task scheduler that is configured 100% by code.

By leveraging Coravel's ease-of-use with the simplicity of .NET Core's worker service project template, I'll show you how easily and quickly you can build a small back-end console application that will run your scheduled background jobs!

# Worker Service Template

First, create an empty folder to house your new project.

Then run:

`dotnet new worker`

Your worker project is all set to go! 🤜🤛

Check out _Program.cs_ and you'll see this:

```csharp
public static void Main(string[] args)
{
    CreateHostBuilder(args).Build().Run();
}

public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureServices(services =>
        {
            services.AddHostedService<Worker>();
        });
```

# Configuring Coravel

Let's add Coravel by running `dotnet add package coravel`.

Next, in _Program.cs_, we'll modify the generic code that was generated for us and configure Coravel:

```csharp
public static void Main(string[] args)
{
    IHost host = CreateHostBuilder(args).Build();
    host.Services.UseScheduler(scheduler => {
        // We'll fill this in later ;)
    });
    host.Run();
}

public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureServices(services =>
        {
            services.AddScheduler();
        });
};
```

Since Coravel is a native .NET Core set of tools, it _just works™_ with zero fuss!

# Adding An Invocable

One of Coravel's fundamental concepts are [Invocables](https://docs.coravel.net/Invocables/).

Each invocable represents a self-contained job within your system that Coravel leverages to make your code much easier to write, compose and maintain.

Next then, create a class that implements `Coravel.Invocable.IInvocable`:

```csharp
public class MyFirstInvocable : IInvocable
{
    public Task Invoke()
    {
        Console.WriteLine("This is my first invocable!");
        return Task.CompletedTask;
    }
}
```

Since we are going to simulate some async work, we'll just log a message to the console and then return `Task.CompletedTask` to the caller.

# Scheduling Your Invocable

Here's where Coravel really shines 😉. 

Let's schedule our new invocable to run every 5 seconds. Inside of our _Program.cs_ main method we'll add:

```csharp
host.Services.UseScheduler(scheduler => {
    // Yes, it's this easy!
    scheduler
        .Schedule<MyFirstInvocable>()
        .EveryFiveSeconds();
});
```

Don't forget to register your invocable with .NET Core's service container:

```csharp
.ConfigureServices(services =>
{
    services.AddScheduler();
    // Add this 👇
    services.AddTransient<MyFirstInvocable>();
});
```

In your terminal, run `dotnet run`.

You should see the output in your terminal every five seconds!

# Real-World Invocable

Sure, writing to the console is great - but you are going to be making API calls, database queries, etc. after all.

Let's modify our invocable so that we can do something more interesting:

```csharp
public class SendDailyReportEmailJob : IInvocable
{
    private IMailer _mailer;
    private IUserRepository _repo;

    public SendDailyReportEmailJob(IMailer mailer, IUserRepository repo)
    {
        this._mailer = mailer;
        this._repo = repo;
    }

    public async Task Invoke()
    {
        var users = await this._repo.GetUsersAsync();

        foreach(var user in users)
        {
            var mailable = new DailyReportMailable(user);
            await this._mailer.SendAsync(mailable);
        }
    }
}
```

Since this class will hook into .NET Core's service container, all the constructor dependencies will be injected via dependency injection.

If you wanted to build a lightweight background application that processes and emails daily reports for all your users then this might be a great option.

# Configuring As A Windows Service

While beyond the scope of this article, you can take a look at how .NET Core 3 will [allow configuring your worker as a windows service](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/windows-service?view=aspnetcore-3.0&tabs=visual-studio-code#app-configuration).

And, apparently, [there's upcoming support for systemd too!](https://github.com/aspnet/Extensions/pull/1804)

# Conclusion

What do you guys think about .NET Core's worker services? 

I find they are so easy to get up-and-running. Coupled with the accessibility designed into Coravel, I find these two make an awesome pair for doing some cool stuff!

All of Coravel's features can be used within these worker services - such as [queuing tasks](https://docs.coravel.net/Queuing/), [event broadcasting](https://docs.coravel.net/Events/), [mailing](https://docs.coravel.net/Mailing/), etc.

One thing I'd love to try is to integrate [Coravel Pro](https://www.pro.coravel.net/) with a worker service. One step at a time though 🤣.

# Keep In Touch

Don't forget to connect with me on:

- [Twitter](https://twitter.com/jamesmh_dev)
- [LinkedIn](https://www.linkedin.com/in/jamesmhickey/)

You can also find me at my web site [www.jamesmichaelhickey.com](https://www.jamesmichaelhickey.com).

# Navigating Your Software Development Career Newsletter

An e-mail newsletter that will help you level-up in your career as a software developer! Ever wonder:

✔ What are the general stages of a software developer?
✔ How do I know which stage I'm at? How do I get to the next stage?
✔ What is a tech leader and how do I become one?
✔ Is there someone willing to walk with me and answer my questions?

Sound interesting? [Join the community!](https://eepurl.com/gdIV5X)


