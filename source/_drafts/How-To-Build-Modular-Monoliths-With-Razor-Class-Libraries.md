---
title: How To Build Modular Monoliths With Razor Class Libraries
tags: microservices, domain-driven design, modular monoliths
---

**Microservices are all the rage now.** But, many are (finally) realizing that it's not for everyone. Most of us aren't at the scale of Netflix, LinkedIn, etc. and therefore don't have the organizational manpower to offset the overhead of a full-blown microservices architecture.

An alternative to a full-blown microservices architecture that's been getting a lot of press lately is called "modular monoliths."

<!-- more -->

> Shouldn't well-written monoliths be modular anyways?

Sure. But modular monoliths are done in a **very intentional** way that usually follows a domain-driven approach.

## Why Modular Monoliths?

Domain-driven practitioners understand that the biggest benefits microservices give us (loose coupling, code ownership, etc.) can be had in a well-designed modular monolith. By leveraging the idea of bounded contexts, we can treat each context as it's own isolated application. 

Yet, instead of hosting each context as an independent process (like with microservices), we can extract each bounded context as a module within a larger system or web of modules. 

For example, each module might be a .NET project/assembly. These assemblies would be combined at run-time and hosted within one main process.

Compared to microservices, the benefits of modular monoliths include:

- In-memory communication between contexts are more performant and reliable than doing it on the network
- A much simpler deployment process for smaller teams
- Retain boundaries and ownership around specific contexts
- Simplified upgrades to contexts/services
- Modules are "ready" to be extracted as services if needed

I see modular monoliths as a step within the potential evolution of a system's architecture:

![spectrum](/img/architecture/ArchitectureSpectrum.png)

But for domain-driven approaches, starting by building a modular monolith might makes the most sense.

## How Can I Build Them?

There are many ways to build modular monoliths.

For now, I want to show one way that's unique to .NET Core using a new feature of .NET Core called Razor Class Libraries. 

This is a simpler way to get started with this architecture than what you might have seen elsewhere.

.NET Core introduced [Razor Class Libraries](https://docs.microsoft.com/en-us/aspnet/core/razor-pages/ui-class?view=aspnetcore-3.0&tabs=visual-studio) as libraries that not only hold application logic (like any old class library) but full UIs too!

For some applications, and even for certain bounded contexts, this might make things much easier than other methods of building modular monoliths.

## Life Insurance Application

I recently wrote [an article](https://www.blog.jamesmichaelhickey.com/DDD-Use-Case-Life-Insurance-Platform/) about using some domain-driven approaches to think about and re-design an insurance selling platform I once worked on.

To keep things simple for now, let's imagine we've determined two bounded contexts from this domain:

- Insurance Application
- Medical Questions

The specific details are not so important since the remainder of this article will look at implementation and code.

## Creating Our Skeleton

Let's implement the skeleton for building this as a modular monolith and use razor class libraries as a way to implement our 

First thing's first - create a new root host process:

`dotnet new mvc -o Host`

Next, we'll create a modules folder within our solution (`Modules`) and then create our two modules as razor class libraries:

`dotnet new razorclasslib -o Modules/InsuranceApplication`

`dotnet new razorclasslib -o Modules/MedicalQuestions`

![folder structure](/img/razormodules/folders1.png)

Next, we'll reference our modules from the host project.

From within the host project:

`dotnet add reference ../Modules/InsuranceApplication/InsuranceApplication.csproj`

`dotnet add reference ../Modules/MedicalQuestions/MedicalQuestions.csproj`

## Building Our First Module



