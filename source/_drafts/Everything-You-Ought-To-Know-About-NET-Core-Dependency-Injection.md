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

TLDR; Dependency injection is baked into .NET Core for a reason: it generally promotes good coding practices and offers developers tools to build maintainable, modular and testable software.

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

Let's quickly run through some of the more proper and technical terms.

## Service Provider

When we refer to the "DI system" we are really talking about the Service Provider.

In other frameworks or DI systems this is also called a _Service Container_.

This is the object that holds the configuration for all the DI stuff.

It's also what will ultimately be "asked" to create new objects for us. And therefore, it's what figures out what dependencies each service require at runtime.

## Binding

When we talk about binding, we just mean that type `A` is mapped to type `B`. 

In our example about the `Car` scenario, we would say that `IEngine` is bound to `HondaEngine`. 

When we ask for a dependency of `IEngine` we are returned an instance of `HondaEngine`.

## Resolving

Resolving refers to the process of figuring out what dependencies are required for a particular service.

Using the example above with the `CreateUser` use case, when the Service Provider is asked to inject an instance of `CreateUser` we would say that the provider is "resolving" that dependency.

Resolving involves figuring out the entire tree of dependencies:

- `CreateUser` requires an instance of `IUserRepository`
- The provider sees that `IUserRepository` is bound to `UserRepository`
- `UserRepository` requires an instance of `ApplicationDbContext`
- The provider see that `ApplicationDbContext` is available (and bound to the same type).

Figuring out that tree of cascading dependencies is what we call "resolving a service."

## Scopes

Generally termed scopes, or otherwise called service lifetimes, this refers to whether a service is shortly lived or long living.

For example, a singleton (as the pattern is defined) is a service that will always resolve to the same instance every time.

Without understanding what scopes are you can run into some really weird errors. 😜

The .NET Core DI system has 3 different scopes:

### Singleton

```csharp
services.AddSingleton<IAlwaysExist, IAlwaysExist>();
```

Whenever we resolve `IAlwaysExist` in an MVC controller constructor, for example, it will always be the exact same instance.

_As a side note: This implies concerns around thread-safety, etc. depending on what you are doing._

### Scoped

```csharp
services.AddScoped<IAmSharedPerRequests, IAmSharedPerRequests>();
```

Scoped is the most complicated lifetime. We'll look at it in more detail later. 

To keep it simple for now, it means that within a particular HttpRequest (in an ASP .NET Core application) the resolved instance will be the same.

Let's say we have service `A` and `B`. Both are resolved by the same controller:

```csharp
public SomeController(A a, B b)
{
    this._a = a;
    this._b = b;
}
```

Now imagine `A` and `B` both rely on service `C`. 

If `C` is a scoped service, and since scoped services resolve to the same instance for the same HttpRequest, both `A` and `B` will have the exact same instance of `C` injected.

Hopefully that's not too confusing. 😜

### Transient

```csharp
services.AddTransient<IAmAlwaysADifferentInstance, IAmAlwaysADifferentInstance>();
```

Transient services are always an entirely new instance when resolved. 

Given this example:

```csharp
public SomeController(A a, A anotherA)
{
}
```

Variables `a` and `anotherA` would be each resolved by the service provider (let's say, as transient services). They will be different instances of type `A` since they are each resolved separately as transient.

_Note: Given the same example, if `A` was a scoped service then variables `a` and `anotherA` would be the same instance. The same goes if `A` was a singleton service._

_However, in the next HttpRequest, if `A` was scoped then `a` and `anotherA` in the next request would be different from the instances in the first request._

_If `A` was a singleton, then variables `a` and `anotherA` in **both** Http requests would reference the same single instance._

## Scope Issues

There are issues that arise when using differently scoped services who are trying to depend on each other.

### Circular Dependencies

Just don't do it. It doesn't make sense 😜

```csharp
public class A
{
    public A(B b) { }
}

public class B
{
    public B(A a){ }
}
```

### Singletons + Transitive Services

A singleton, again, lives "forever". It's always the same instance. 

Transitive services, on the other hand, are always a different instance when requested - or resolved.

So here's an interesting question: When a singleton depends on a transitive dependency **how long does the transitive dependency live?**

The answer is **forever**. More specifically, **as long as it's parent lives.**

Since the singleton lives forever so will all of it's child objects that it references.

This isn't necessarily **bad**. But it could introduce weird issues when you don't understand what this setup implies.

#### Thread-Safety

Perhaps you have a transitive service - let's call it `ListService` that (because it's transitive) isn't thread safe. 

`ListService` has a list of stuff and exposes methods to `Add` and `Remove` those items.

Now, you started using `ListService` inside of a singleton as a dependency.

That singleton will be re-used **everywhere**. That means, **on every HTTP Request**. Which implies **on many many different threads.**

Since the singleton accesses/uses `ListService`, and `ListService` isn't thread safe - big problems!

### Singletons + Scoped Services

Lets assume now that `ListService` is a scoped service.

If you try to inject a scoped service into a singleton what will happen?

**.NET Core will blow up and tell you that you can't do it!**

Remember that scoped services live for as long as an HTTP request? 

But, remember how I said it's actually more complicated than that?...

#### How Scoped Services Really Work

Under the covers .NET Core's service provider exposes a method `CreateScope`.

_Note: Alternatively, you can uses `IServiceScopeFactory` (injected) and use the same method `CreateScope`. We'll look at this later_😉

`CreateScope` creates a "scope" that implements the `IDisposable` interface. It would be used like this:

```csharp
using(var scope = serviceProvider.CreateScope())
{
    // Do stuff...
}
```

The service provider also exposes methods for resolving services: `GetService` and `GetRequiredService`.

The difference between them is that `GetService` returns null when a service isn't bound to the provider, and `GetRequiredService` will throw an exception.

So, a scope might be used like this:

```csharp
using(var scope = serviceProvider.CreateScope())
{
    var provider = scope.ServiceProvider;
    var resolvedService = provider.GetRequiredService(someType);
    // Use resolvedService...
}
```

When .NET Core begins an HTTP request under the covers it'll do something like that. It will resolve the services that your controller may need, for example, so you don't have to worry about the low-level details.

In terms of injecting services into ASP controllers - scoped services are basically attached to the life of the HTTP request.

But, we can create our own services (which would then be a form of the Service Locator pattern - more on that later)!

#### Multiple Service Providers

Notice how each scope has it's own `ServiceProvider`? What's up with that?

The DI system has **multiple Service Providers.** Woah 🤯

Singletons are resolved from a root service provider (which exists for the lifetime of your app). In a matter of speaking, the root provider **is not scoped**.

But, anytime you create a scope - you get a **scoped** service provider! This scoped provider will still be able to resolve singleton services, but by proxy they come from the root provider.

Both providers can create transitive services.

Here's the rundown of what we just learned:

- Singleton services are always resolvable (from root provider or by proxy)
- Transitive service are always resolvable (from root provider or by scoped)
- Scoped services require a scope and therefore a scoped service provider to be resolvable

So what happens when we try to resolve a scoped service from the root provider (a non-scoped provider)?... Boom 🔥

#### Back To Our Topic

All that to say that scoped services **require a scope to exist**.

Singletons are resolved by the root provider.

Since the root provider has no scope (it's a "global" provider in a sense) - it just doesn't make sense to inject a scoped service into a singleton.

### Scoped + Transitive Services

What about a scoped service who relies on a transitive service?

In practice it'll work. But, for the same reasons as using a transitive services inside a singleton, it may no behave as you expect.

The transitive service that is used by the scoped service will live as long as the scoped service.

Just be sure that makes sense within you use-case.

## Dependency Injection For Libraries

As library authors we sometimes want to provide native-like tools. For example, with [Coravel](https://github.com/jamesmh/coravel) - I wanted to make the library integrate seamlessly with the .NET Core DI system.

How do we do that?

### IServiceScopeFactory

As mentioned in passing, .NET Core provides a utility for creating scopes. This is useful for library authors.

Instead of grabbing an instance of `IServiceProvider`, library authors probably should use `IServiceScopeFactory`.

Why? Well, remember how the root service provider cannot resolve scoped services? What if your library needs to do some "magic" around scoped services? Oops!

[Coravel](https://github.com/jamesmh/coravel), for example, needs to resolve certain types from the service provider in certain situations (like instantiating [invocable classes](https://github.com/jamesmh/coravel/blob/master/Docs/Invocables.md)).

Entity Framework Core contexts are scoped, so doing things such as performing database queries inside your library (on behalf of the user/developer) is something you may want to do.

This is something that [Coravel Pro](https://www.pro.coravel.net/) does - execute queries from the user's EF Core context automatically under-the-covers.

### Service Locator Pattern

In general, the service locator pattern is not a good practice. This is when we ask for a specific type from the service provider manually.

```csharp
using(var scope = serviceProvider.CreateScope())
{
    var provider = scope.ServiceProvider;
    var resolvedService = provider.GetRequiredService(someType);
    // Use resolvedService...
}
```

However, for cases like mentioned above, it is what we need to do - grab a scope, resolve services and do some "magic".

This would be akin to how .NET Core prepares a DI scope and resolves services for your ASP .NET Core controllers.

It's not bad because it's not "user code" but "utility code".

# Conclusion

We looked at some reasons behind why dependency injection is a useful tool at our disposal.

It helps to promote

- Code testability
- Code reuse through composition
- Code readability

Next we look at how dependency injection in .NET Core is used, and some of the lower-level aspects of how it works.

In general, we found that **problems arise when services rely on other services who have a shorter lifetime.**

Finally we looked at how .NET Core provides library authors with some useful tools that can help integration with .NET Core's DI system seamless.

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
