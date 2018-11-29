---
title: Everything You Ought To Know About .NET Core Dependency Injection
tags:
    - Dependency Injection
    - .NET Core
    - .NET Core Dependency Injection
---

Is it necessary to understand what dependency injection is to use .NET Core? Should you know how to use it?

If you go to the [official docs for ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-2.1) you'll find that this topic is under the "fundamentals" area.

Is that true - is it really **fundamental?**

# What I've Learned From Building Coravel: Part 3

This is part 3 of an ongoing series. The other editions are:

- [Part 1: What I've Learned So Far Building Coravel (Open Source .NET Core Tooling)](https://www.blog.jamesmichaelhickey.com/What-I-ve-Learned-So-Far-Building-Coravel-Open-Source-NET-Core-Tooling/)
- [Part 2: Fluent APIs Make Developers Love Using Your .NET Libraries](https://builtwithdot.net/blog/fluent-apis-make-developers-love-using-your-net-libraries)

As you guessed, this article will go over some things I've learned about DI in .NET Core, along with my suggestions for what you should know. 😊

However, to begin - I want to explore DI for those who may not be too familiar with dependency injection to begin with. We'll start with the basics and more toward some more advanced scenarios.

If you already know what DI is, and how to use interfaces to mock your classes and test them, etc. then you can move onto the "What You Should Know About Using .NET Core Dependency Injection" section.

Yes, this is going to be a long one. Get ready. 😎

# What Is Dependency Injection?

If you aren't familiar with DI, it pretty much just refers to passing dependencies into your objects as an external arguments.

This can be done via an object's constructor or method.

```csharp
// Argument "dep" is "injected" as a dependency.
public MyClass(ExternalDependency dep)
{
    this._dep = dep;
}
```

Dependency injection then is, at a fundamental level, just passing dependencies as arguments. That's it. That's all.

Well... if that was **really** all of what DI is - I wouldn't be writing this. 😜

# Why Should We Pass Dependencies As Arguments?

Why would you want to do this? A few reasons:

- Promotes splitting logic into multiple smaller classes and/or structures
- Promotes code testability
- Promotes using abstractions that allow a more modular code structure in general

Let's look briefly at the idea that this promotes testability (which in turn affects all the other points mentioned).

Why do we test code? **To make sure our system behaves properly.** 

This means that **you can trust your code**.

With no tests, **you can't really trust your code**.

I discuss this in more detail in another blog post about [Refactoring Legacy Monoliths](https://www.blog.jamesmichaelhickey.com/refactoring-game-plan-and-tips/) - where I discuss some refactoring techniques around this issue.

TLDR; **You want to be able to trust your code. Use tests.**

# What Is Dependency Injection (Revisited)

Of course, DI is more than "just passing in arguments." Dependency injection is a mechanism where the runtime (let's say - .NET Core) will automatically pass (inject) required dependencies into your classes.

Why would we ever need that?

Look at this simple class:

```csharp
public class Car
{
    public Car(Engine engine)
    {
        this._engine = engine;
    }
}
```

What if, somewhere else, we needed to do this:

```csharp
Car ferrari = new Car(new Engine());
```

Great. What if we wanted to test this `Car` class?

The problem is **in order to test `Car` you need `Engine`.** This is a "hard" dependency, if you will.

This means that these classes are tightly tied together. In other words, tightly coupled. Those are bad words.

We want loosely coupled classes. This makes our code more modular, generalized and easier to test (which means **more trust** and **more flexibility**).

# Quick Look At Testing

Some common techniques when testing are to use "mocks". A mock is just a stubbed-out class that "pretends" to be a real implementation.

We can't mock concrete classes. But, we can mock interfaces!

Let's change our `Car` to rely on an interface instead:

```csharp
public class Car
{
    public Car(IEngine engine)
    {
        this._engine = engine;
    }
}
```

Cool! Let's test that:

```csharp
// Mock code configuration would be here.
// "mockEngine" would just have dummy methods since we just want to test how the 
// Car class works. 
// 
// We can issue other tests specifically for the Engine later.
Car ferrari = new Car(mockEngine);

Assert.IsTrue(ferrari.IsFast());
```

So now we are testing the `Car` class **without a hard dependency on `Engine`**. 👍

# A Quick Look At Modularization

I had mentioned that using DI allows your code to be modular. Well, it's not really DI that does, but the technique above (relying on interfaces).

Compare these two examples:

```csharp
Car ferrari = new Car(new FastEngine());
```

```csharp
Car civic = new Car(new HondaEngine());
```

Since we are relying on interfaces, we have way more flexibility as to what kinds of cars we can build!

# Avoiding Class Inheritance

Another benefit is that you **don't need to use class inheritance.** 

This is something I see abused all the time. So much so that I do my best to "never" use class inheritance.

It's hard to test, it's hard to understand and it's usually leads to building an incorrect model anyways since it's so hard to change after-the-fact.

99% of the time there are better ways to build your code using patterns like this - which rely on abstractions rather than tightly coupled classes.

And yes - class inheritance is **the most** highly coupled relationship you can have in your code! (But that's another blog post 😉)

# Using Dependency Injection In .NET Core

The example above highlights why we need DI. 

Perhaps in a web app that is designed for Honda we would need to use the `HondaEngine`, but for Ford we need the `FordEngine`?

Dependency injection allows us to "bind" a specific class to be used globally. 

At runtime, we rely on the DI system to create new instances of these objects for us. All the dependencies are handled automatically.

In .NET Core, you might do something like this to tell the DI system what classes we want to use when asking for certain interfaces, etc.

```csharp
// Whenever the type 'Car' is asked for we get a new instance of the 'Car' type.
services.AddTransient<Car,Car>(); 
// Whenever the type 'IEngine' is asked for we get a new instance of the concrete 'HondaEngine' type.
services.AddTransient<IEngine, HondaEngine>();
```

`Car` relies on `IEngine`. 

When the DI system tries to "build" (instantiate) a new `Car` it will first grab a `new HondaEngine()` and then inject that into the `new Car()`.

Whenever we need a `Car` .NET Core's DI system will automatically rig that up for us! All the dependencies will cascade.

So, In an MVC controller we might do this:

```csharp
public CarController(Car car)
{
    this._car = car; // Car will already be instantiated by the DI system using the 'HondaEngine' as the engine.
}
```

# A Real World Example

Alright - the car example was simple. That's to get the basics down. Let's look at a more realistic scenario.

Get ready. 😎

We have a use case for creating a new user in our app:

```csharp
public class CreateUser
{    
    // Filled out later...
}
```

That use case needs to issue some database queries to persist new users. 

In order to make this testable - and make sure that we can test our code **without requiring the database as a dependency** - we can use the technique already discussed:

```csharp
public interface IUserRepository
{
    public Task<int> CreateUserAsync(UserModel user);
}
```

And the concrete implementation that will hit the database:

```csharp
public class UserRepository : IUserRepository
{
    private readonly ApplicationDbContext _dbContext;

    public UserRepository(ApplicationDbContext dbContext) =>
        this._dbContext = dbContext;

    public async Task<int> CreateUserAsync(UserModel user)
    {
        this._dbContext.Users.Add(user);
        await this._dbContext.SaveChangesAsync();
    }
}
```

Using DI, we would have something like this:

```csharp
services.AddTransient<CreateUser, CreateUser>();
services.AddTransient<IUserRepository, UserRepository>();
```

Whenever we have a class that needs an instance of the `IUserRepository` the DI system will automatically build a `new` `UserRepository` for us. 

The same can be said for `CreateUser` - a new `CreateUser` will be given to us when asked (along with all of it's dependencies already injected).

Now, in our use case we do this:

```csharp
public class CreateUser
{
    private readonly IUserRepository _repo;

    public CreateUser(IUserRepository repo) =>
        this._repo = repo;

    public async Task InvokeAsync(UserModel user)
    {
        await this._repo.CreateUserAsync(user);
    }    
}
```

In an MVC controller, we can "ask" for the `CreateUser` use case:

```csharp
public class CreateUserController : Controller
{
    private readonly CreateUser _createUser;

    public CreateUserController(CreateUser createUser) =>
        this._createUser = createUser;

    [HttpPost]
    public async Task<ActionResult> Create(UserModel userModel)
    {
        await this._createUser.InvokeAsync(userModel);
        return Ok();
    }    
}
```

The DI system will automatically:
- Try to create a new instance of `CreateUser`.
- Since `CreateUser` depends on the `IUserRepository` interface, the DI system will next look to see if there is a type "bound" to that interface. 
- Yes - it's the concrete `UserRepository`. 
- Create a new `UserRepository`.
- Pass that into a new `CreateUser` as the implementation of it's constructor argument `IUserRepository`.

Some benefits that are obvious:

- Your code is much more modular and flexible (as mentioned)
- Your controllers etc. (whatever is using DI) become way simpler and easy to read.

# Real World Testing

And the final benefit, again, we can test this **without needing to hit the database**.

```csharp
// Some mock configuration...
var createUser = new CreateUser(mockUserRepositoryThatReturnsMockData);
int createdUserId = await createUser.InvokeAsync(dummyUserModel);

Assert.IsTrue(createdUserId == expectedCreatedUserId);
```

This makes for:

- Fast testing (no database)
- Isolated testing (only focusing on testing the code in `CreateUser`)

# What You Should Know About Using .NET Core Dependency Injection



# Conclusion.

<hr />

<div style="padding:20px; border-radius:6px; background-color: #efefef; margin-bottom:50px">
    <h1 class="margin-bottom:0"><img src="https://www.pro.coravel.net/img/logo.png" style="width:47px;margin-top:-2px;border-radius:6px;margin-right:20px" /> Coravel Pro
</h1>
I've been building [Coravel Pro](https://www.pro.coravel.net/) which is a suite of professional admin backend tools that help you schedule and manage your backend admin jobs.

<strong>Schedule your jobs with database persistence</strong> - so your dev schedules don't bleed into your production schedules!

<strong>Execute backend jobs</strong> by literally clicking one button!

Easily configure a <strong>metrics dashboard!</strong>

Quickly build-out <strong>tabular reports</strong> that integrate seamlessly with your Entity Framework Core data!
    <div class="text-center">
        <a href="https://www.pro.coravel.net/">
            <button class="btn btn-sign-up" style="margin-top:0;margin-bottom:0">Take A Look At Coravel Pro!</button>
        </a>
    </div>
</div>

## You Might Enjoy

- [What Makes .NET Core So Special?](https://www.blog.jamesmichaelhickey.com/What-Makes-NET-Core-So-Special-Why-You-Should-Use-NET-Core/)
- [What I've Learned So Far Building Coravel (Open Source .NET Core Tooling)](https://www.blog.jamesmichaelhickey.com/What-I-ve-Learned-So-Far-Building-Coravel-Open-Source-NET-Core-Tooling/)
- [Fluent APIs Make Developers Love Using Your .NET Libraries](https://builtwithdot.net/blog/fluent-apis-make-developers-love-using-your-net-libraries)

## Keep In Touch

Don't forget to connect with me on [twitter](https://twitter.com/jamesmh_dev) or [LinkedIn](https://www.linkedin.com/in/jamesmhickey/)!

I also have an e-mail letter where I'll give you tips, stories and links to **help ambitious and passionate developers become tech leaders.** I'll also give you updates about stuff that I've been working on ;)

[Subscribe if you haven't already!](https://tinyletter.com/jamesmh)
