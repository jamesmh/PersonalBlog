---
title: How To Build Modular Monoliths With .NET Core Razor Class Libraries
tags: microservices, domain-driven design, modular monoliths
---

**Microservices are all the rage now.** 

But, many are (finally) realizing that it's not for everyone. Most of us aren't at the scale of Netflix, LinkedIn, etc. and therefore don't have the organizational manpower to offset the overhead of a full-blown microservices architecture.

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

But for domain-driven approaches, starting with a modular monolith might make the most sense.

## How Can I Build Them?

There are many ways to build modular monoliths.

For now, I want to show one way that's unique to .NET Core: using a new feature of .NET Core called razor class libraries. 

This is a simpler way to get started with this architecture than what you might have seen elsewhere.

.NET Core introduced [razor class libraries](https://docs.microsoft.com/en-us/aspnet/core/razor-pages/ui-class?view=aspnetcore-3.0&tabs=visual-studio) as libraries that not only hold application logic (like any old class library) but full UIs too!

For some applications, this might make things much easier than other methods of building modular monoliths.

## Life Insurance Application

I recently wrote [an article](https://www.blog.jamesmichaelhickey.com/DDD-Use-Case-Life-Insurance-Platform/) about using some domain-driven approaches to think about and re-design an insurance selling platform I once worked on.

To keep things simple for now, let's imagine we've determined two bounded contexts from this domain:

- Insurance Application
- Medical Questionnaire

The specific details are not so important since the remainder of this article will look at implementation and code.

## Creating Our Skeleton

Let's implement the skeleton for our modular monolith and use razor class libraries as a way to implement our bounded contexts.

First thing's first - create a new root host process:

`dotnet new webapp -o HostApp`

Next, we'll create a modules folder within our solution (`Modules`) and then create our two modules as razor class libraries:

`dotnet new razorclasslib -o Modules/InsuranceApplication`

`dotnet new razorclasslib -o Modules/MedicalQuestions`

![folder structure](/img/razormodules/folders1.png)

Next, we'll reference our modules from the host project.

From within the host project:

`dotnet add reference ../Modules/InsuranceApplication/InsuranceApplication.csproj`

`dotnet add reference ../Modules/MedicalQuestions/MedicalQuestions.csproj`

## Building Our First Module

Navigate to the Insurance Application module at `Modules/InsuranceApplication`.

Your razor class library will have some sample Blazor files - you can remove those (Sorry - this is not about Blazor!)

Let's build a couple Razor Pages that will be used as this bounded context's UI:

`dotnet new page -o Areas/InsuranceApplication/Pages -n ContactInfo`

`dotnet new page -o Areas/InsuranceApplication/Pages -n InsuranceSelection`

> You might have to go into your razor pages and adjust the generated namespaces. In my example, I changed them to `InsuranceApplication.Areas.InsuranceApplication.Pages`.

## Oops! It Doesn't Work!

Navigate back to the `HostApp` web project and try to build it. It will fail.

Our razor class libraries are trying to use razor pages - which is a web function. This requires referencing the appropriate reference assemblies.

Before .NET Core 3.0, we would have added a reference to some extra NuGet packages like `Microsoft.AspNetCore.Mvc`, etc. As of .NET Core 3.0, many of these ASP related packages are actually included in the .NET Core SDK.

For some projects, this breaking change can cause issues and confusion! For more details, check out Andrew Lock's [in-depth look at this issue.](https://andrewlock.net/converting-a-netstandard-2-library-to-netcore-3/)

For our scenario, we'll have to change the project files of our two razor class libraries to the following:

```xml
<Project Sdk="Microsoft.NET.Sdk.Razor">

  <PropertyGroup>
    <TargetFramework>netcoreapp3.0</TargetFramework>
    <AddRazorSupportForMvc>True</AddRazorSupportForMvc>
  </PropertyGroup>

  <ItemGroup>
    <FrameworkReference Include="Microsoft.AspNetCore.App" />
  </ItemGroup>

</Project>
```

> Yes, I removed all the boilerplate Blazor code - sorry!

## It's Alive!

You should now go into your razor pages (inside the `InsuranceApplication` project) and add some dummy HTML.

Ex.

```html
@page
@model InsuranceApplication.Areas.InsuranceApplication.Pages.ContactInfoModel
@{
}

<h1>Insurance Contact Info</h1>
```

Try running your host application and navigate to `/InsuranceApplication/ContactInfo` and `/InsuranceApplication/InsuranceSelection`.

Both pages should display. Cool!

## Build The UI Flow

### Beginning The Insurance Application

Let's look at building out a basic flow between our two modules.

First, in the `HostApp` project, in your `Pages/Index.cshtml` file add the following HTML:

```html
<a href="/InsuranceApplication/ContactInfo">Begin Your Application!</a>
```

That will give us a link to click from our home screen to begin our flow through our application logic.

![folder structure](/img/razormodules/flow1.png)

### Inside The Insurance Application Module

In your `ContactInfo.cshtml` file within the `InsuranceApplication` project, insert the following:

```html
@page
@model InsuranceApplication.Areas.InsuranceApplication.Pages.ContactInfoModel
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

<h1>Insurance Contact Info</h1>

<form method="post">
    <label>Email:</label>
    <input asp-for="EmailAddress" required type="email" />

    <button type="submit">submit</button>
</form>
```

In `ContactInfo.cshtml.cs`, add the following:

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace InsuranceApplication.Areas.InsuranceApplication.Pages
{
    public class ContactInfoModel : PageModel
    {
        [BindProperty]
        public string EmailAddress { get; set; }

        public IActionResult OnPostAsync()
        {
            Console.WriteLine($"Email Address is {this.EmailAddress}.");
            return RedirectToPage("./InsuranceSelection");
        }
    }
}
```

This will allow us to enter an email address and then move on to the next page in our flow.

![folder structure](/img/razormodules/flow2.png)

In the `InsuranceSelection.cshtml` page - just add an link to keep things simple for now:

```html
<a href="/MedicalQuestionnaire/Questions">Next<a>
```

### Medical Questionnaire Module

You might have noticed that we never created any razor pages in the Medical Questionnaire module. Let's do that.

Navigate to the root of the `MedicalQuestions` project and enter the following from your terminal:

`dotnet new page -o Areas/MedicalQuestionnaire/Pages -n Questions`

> Again, you might need to adjust the generated namespaces.

Within the `Questions.cshtml` file, add a link:

```html
<h1>Medical Questionnaire</h1>

<a href="/">Finish<a>
```

Now, you can run through the entire UI using multiple modules behind the scenes!

## Other Considerations

### Business Logic

If each module is a DDD bounded context then it can:

- Use it's own isolated database
- Have it's own DDD business logic
- Communicate to other bounded contexts by using messaging and domain events
- etc.

### Deployment

You can also deploy each library as a NuGet package and allow individual teams have ownership of specific modules.

### Compositional UIs

Using [razor components](https://docs.microsoft.com/en-us/aspnet/core/blazor/components?view=aspnetcore-3.0), [view components](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/view-components?view=aspnetcore-3.0), etc. you can allow each module to define it's own set of components (UI and backend logic included).

These can be plugged into a composite UI that might be housed from the host application.

### Next Steps

For more around what you can do with razor libraries, defining layouts, and more specifics [check out this article.](https://www.learnrazorpages.com/advanced/razor-class-library)