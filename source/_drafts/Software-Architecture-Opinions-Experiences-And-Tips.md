---
title: 'Software Architecture: Opinions, Experiences And Tips'
tags:
---

One of the challenges I faced about 2 years into my career was being named the lead for building out the web portion of a white-label product that would allow life insurance companies potential customers to apply for their insurance online.

Most of the core business rules were already coded and existed as a class library. 

That's a long story in itself...

One of the specific challenges, experiences and learnings that came out from that project were around code organization and architecture.

Moving forward to other projects I've been on with other companies, I've had the opportunity to learn and validate opinions and thoughts around what I've found worked well and what didn't.

I'd like to share some of my current approaches and why I find they work (or don't work).

## A Note About MVC

Most web applications are built created using an MVC-like structure. And that's it.

This might work well for smaller projects, but for non-trivial business applications not so much.

[I wrote about this previously](https://builtwithdot.net/blog/changing-how-your-code-is-organized-could-speed-development-from-weeks-to-days).

In a nutshell, our systems and applications should be organized to match the business/domain we are working in. MVC is a useful pattern _within_ these sub-systems that we create (whether we are calling this sub-systems domains, sub-domains, features, modules, etc. is another topic.)

With that out of the way, let's move on!

## Brief Look At Software Architecture

There are many kinds of architectures and high-level patterns available for us to use as developers and architects. Some are useful in very specific contexts, and others are more general purpose and work well for most cases.

Some of the patterns that I've found most helpful in my own search for the best way to organize code and projects, how to separate different domains or features, how to allow communication between domains, etc. include:

- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Vertical Slices](https://jimmybogard.com/vertical-slice-architecture/)
- [CQRS](http://codebetter.com/gregyoung/2010/02/16/cqrs-task-based-uis-event-sourcing-agh/)
- [Domain-Driven Approaches](http://domainlanguage.com/ddd/)
- [Event-Driven Styles](https://docs.microsoft.com/en-us/azure/architecture/guide/architecture-styles/event-driven)

I've tried implementing, for example, a full-on hexagonal architecture similar to Clean Architecture. I've implemented CQRS in various ways.

Throughout my experiences, I've come to my own conclusions about what works best for me and what I find most practical.

## Starting With Context

When starting a new project or looking at improving the software of a business, the first area to delve into has to be around bounded contexts. 

In microservice scenarios, this might be called service boundaries.

Why?

One of the main issues that implementing a well structured architecture is **dealing with complexity.**

By focusing first on the different contexts or main areas of our business, we can tackle each piece separately.

It also means that some more complex contexts can use different techniques or even different inner architectural approaches that work best for those specific business problems.

### Example Scenario

For example, when dealing with building software for a dental practice you might have a few bounded contexts that you've discovered:

- Clients (where you store details about a specific customer/client)
- Payments 
- Appointments (deals with booking appointments and scheduling)
- Dental Work (deals with work done on a client)

If we isolate each of these areas and tackle them separately, some of them might be really simple to implement and can use a simple approach (like CRUD). The clients context would most likely be an example.

On the other side of the spectrum, some contexts are complex. They need more advanced techniques and patterns. More thought and design needs to implement these ones well. The Dental Work context would probably be the most complex example.

![complexity](/img/architecture/complexity.png)

### Tools To Discover Bounded Contexts

As a first step then, discovering bounded contexts is critical. 

The baseline tool in the community is [Event Storming](https://leanpub.com/introducing_eventstorming).

There are other variants of this of course. 

I've found, for my own experiences, that event storming works well whether you are trying to manage a new product/system or even whether you are working in a legacy system and working on improving a piece of it.

This can get really deep! Here are some more resources if interested in getting deeper into this area:

[Finding Service Boundaries: Illustrated In Healthcare](https://youtu.be/RhfyP8pEEc4?t=336)
[Jimmy Bogard Presenting On Real-World Project And Finding Boundaries](https://www.youtube.com/watch?v=U6CeaA-Phqo)
[Finding Service Boundaries: A Practical Guide](https://www.youtube.com/watch?v=tVnIUZbsxWI)

### What About Legacy Applications?

Most talks and articles around bounded contexts apply when creating new systems.

Personally, I've had to do this but also deal with legacy systems that need new features yet cannot "touch" the existing features too much.

I've found [this white paper](http://domainlanguage.com/wp-content/uploads/2016/04/GettingStartedWithDDDWhenSurroundedByLegacySystemsV1.pdf) to be helpful in understanding some ways to approach this in complex legacy scenarios.

At the end of the day, the best approach is to isolate the new features as much as possible from the legacy code.

Not allowed to create a new database? Then just keep your code as separate as possible.

**The worst thing you can do is continue in the methods of building software that are known to produce software that's hard to understand and maintain.**

## Use Cases

One of the biggest epiphanies I had was in reading about [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html). There's a lot to it (there's an [entire book about it](https://amzn.to/2ZHscdK)).

The most helpful idea that I grabbed from this method or architecture is the concept of use cases.

There's a big focus on discovering what the specific use cases in your system or domain are, and modelling those explicitly.

### What We Normally Do

Normally, what I've seen is that developers create classes to do things. Then, inside an MVC controller action, we'll do stuff with those classes.

Sounds fine.

Until you start sharing these classes all over the place. Until you try to find files based on certain scenarios in your business.

### What To Do Instead

Focusing on use cases is way better. 

For example, in the dental office example from above, we might have a use case to model when an appointment is booked for a client.

Instead of creating an `AppointmentController` with a `BookApointment` method, and then doing work in there, we would model this scenario with an explicit class:

```c#
public class BookDentalAppointmentForClient
{
   // ...
}
```

There's more to it than that, but the general concept is pretty simple. 

## It Makes Building Requirements Easier

Another big benefit I've found with this approach is that it gives a really great way to map functional requirements to actual code that is written.

For each bounded context, we might draft up a requirements document that's used to plan and delegate work to developers.

Each major section in this document will be for a specific use case.

Imagine, as a developer, you were given this requirement to start coding:

> #### Book Dental Appointment For Client
>
> **Description**:

> This is when the office clerk will schedule a future appointment with a client.
>
> **Accepts**:
> - clientId
> - assigned dental professional's id (doctor, etc.)
> - UTC datetime of the appointment
> - UTC datetime of the estimated end of the appointment
> - Additional notes for the appointment
>
> **Development Notes**:
>
> Due to the complexity of this scenario, we should use advanced DDD techniques like aggregates. Using value objects for validation of the business rules will be helpful and appropriate also.
>
> Here's a quick sketch of what this value object might look like:
>
> [image here]
>
> **Domain Events Emitted:**
>
> - DentalAppointmentScheduled

If you were working with this use case approach, and additionally a domain-driven approach too, then you'll know right away that:

- You need to create an explicit use case class for this scenario (`BookDentalAppointmentForClient`)
- You'll have to look into creating classes for the DDD aggregate, value objects, etc.
- You know what domain events will be needed to create.

Of course, more details can be added as this is a small sample to show you how approaching requirements this way might help you.

Some of these ideas include:

- Keeping bounded contexts separated is first and foremost required. This gives the flexibility to allow each one to evolve as needed, not affect other BCs when changed and keeps complex BCs isolated and therefore easier to work with.

- Focusing on use cases of the system is a critical piece of the pie. I've found modelling use cases as either Commands or Queries (ala CQRS and vertical slices).

  Use cases are essentially the API that a bounded context will expose. The only way to interact with a BC is through use cases.

  This works fantastic when you need to share certain capabilities of a system between multiple applications (web, API, console, etc.)

- I find focusing on testing only command use cases as a balanced and practical way to work well. This ensures you aren't wasting time testing things that aren't actually in any usage paths, etc.

   Generally, I wouldn't test most queries. For more important uses like queries used in business critical reporting then I would test the query.

- Use cases never expose domain objects to the outside.

- Use cases expose their own specific and isolated contract for what it accepts as input and what it gives as output.

- Keeping bounded contexts in their own assembly seems to work better than not.

- For repository interfaces and implementations, I've found, it usually works best keeping them in the same place. I've found splitting each bounded context into a data, infrastructure, domain, etc. projects causes too much context switching and is unnecessary (until you **do** find it _necessary_).

- Performing all authorization logic within use cases and NOT in the application layer (e.g. web app, web api, etc.). This way you can just use a use case within another application and not have to worry about duplicating authorization filters, etc.

This repo might help to give you some new ideas or raise some interesting questions or ideas about what works well, what doesn't, what contexts might fit this better than others, etc.

And of course, context does matter. Depending on the specific project, company, dev team, etc. I might look at simplifying or implementing some more complex patterns/techniques. But this, I think, is a good starting point.
