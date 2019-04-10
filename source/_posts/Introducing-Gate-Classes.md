---
title: Introducing Gate Classes
tags:
  - 'C#'
  - software design
date: 2019-04-10 16:11:10
---


Have you ever heard of guard clauses? Steve Smith discusses them in one of his Weekly Dev Tips [here](https://www.weeklydevtips.com/004).

Today I want to show you a technique that I've been using which is similar to the guard clause pattern but is used in more advanced scenarios. These scenarios include when you have problems due to external dependencies or complex logic that is required in your guard clauses.

<!-- more -->

With guard clauses, we would take code that has nested conditional logic like this:

```csharp
if(order != null)
{
    if(order.Items != null)
    {
        this.PlaceOrder(order);
    }
    else {
        throw new ArgumentNullException("Order is null");
    }
}
else {
    throw new ArgumentNullException("Order is null");
}
```

Then we would invert the conditional logic and try to "fail fast":

```csharp
if(order?.Items == null)
{
    throw new ArgumentNullException("Order is null");
}

this.PlaceOrder(order);
```

Notice that right-off-the-bat we will try to make the method fail? That's what I mean by "failing fast".

This will keep our code much cleaner, avoid any nested conditions, and be _way_ easier to reason about!

# What If You Have Dependency Baggage?

I love this pattern. 

But, sometimes you might find yourself trying to build a type of guard clause which has certain external dependencies, like a repository or `HttpClient`. Perhaps the logic for the actual guard is quite complex too.

Examples might include determining if:

- A user has proper permissions to view a certain resource in your system
- A potential order is capable of being purchased (in a simple retail system)
- An insurance claim is capable of being approved
- etc.

What I like to do in these cases is use what I've been calling "Gate Classes." They're like guard clauses, but they are classes... Go figure.

Let me show you what I mean.

# Checking If We Can Approve An Insurance Claim

Imagine we are building part of an insurance processing system. 

Next, we have to check whether the claim is able to be approved, and if so, approve it.

Here's our use case (Clean Architecture) or, as some might know it, our Command (CQRS):

```csharp
public class ApproveInsuranceClaimCommand : IUseCase
{
    private readonly IInsuranceClaimRepository _claimRepo;
    private readonly IUserRepository _userRepo;

    public ApproveInsuranceClaimCommand(
        IInsuranceClaimRepository claimRepo, 
        IUserRepository userRepo
    )
    {
        this._claimRepo = claimRepo;
        this._userRepo = userRepo;
    }

    public async Task Handle(Guid claimId, int approvingUserId)
    {
        var user = await this._userRepo.Find(approvingUserId);

        // 1. Figure out if the user has permission to approve this...

        InsuranceClaim claim = await this._claimRepo.Find(claimId);

        // 2. Figure out if the claim is approvable...

        claim.Approve();
        await this._claimRepo.Update(claim);
    }
}
```

For the logic that will go into the comments I made, what if we required **more** repositories to make those checks?

Also, what if other use cases in our system needed to make those same checks?

Perhaps we have another use case called `ApproveOnHoldInsuranceClaimCommand` that will approve an insurance claim that was, for some reason, placed on hold until further documentation was supplied by the customer?

Or, perhaps in other use cases we need to check if users are able to have permission to change a claim?

This will lead to messy code and a lot of copy and pasting!

# The Solution From The Outside

Just like the guard clause refactoring pattern, why don't we do the same thing but convert each guard clause into an entirely new class?

The benefits are that we can use dependency injection to inject any dependencies like repositories, `HttpClient`s, etc. that **only** each gate class will require.

Now, our use case might look like this (keeping in mind that each gate class may do some complex logic inside):

```csharp
public class ApproveInsuranceClaimCommand : IUseCase
{
    private readonly IInsuranceClaimRepository _claimRepo;
    private readonly CanUserApproveInsuranceClaimGate _canUserApprove;
    private readonly CanInsuranceClaimBeApprovedGate _claimCanBeApproved;

    public ApproveInsuranceClaimCommand(
        IInsuranceClaimRepository claimRepo
        CanUserApproveInsuranceClaimGate canUserApprove, 
        CanInsuranceClaimBeApprovedGate claimCanBeApproved
    )
    {
        this._claimRepo = claimRepo;
        this._canUserApprove = canUserApprove;
        this._claimCanBeApproved = claimCanBeApproved;
    }

    public async Task Handle(Guid claimId, int approvingUserId)
    {
        await this._canUserApprove.Invoke(approvingUserId);
        InsuranceClaim claim = await this._claimRepo.Find(claimId);
        await this._claimCanBeApproved.Invoke(claim);
        claim.Approve();
        await this._claimRepo.Update(claim);
    }
}
```

Notice that there's no more need for the `IUserRepository` since it will be handled by the `CanUserApproveInsuranceClaimGate` gate class (and DI).

# Creating A Gate Class

Let's look at how we might build the `CanInsuranceClaimBeApprovedGate` gate class:

```csharp
public class CanInsuranceClaimBeApprovedGate
{
    private readonly IInsuranceClaimAdjusterRepository _adjusterRepo;
    private readonly IInsuranceClaimLegalOfficeRepository _legalRepo;

    public CanInsuranceClaimBeApprovedGate(
        IInsuranceClaimAdjusterRepository adjusterRepo,
        IInsuranceClaimLegalOfficeRepository legalRepo
    )
    {
        this._adjusterRepo = adjusterRepo;
        this._legalRepo = legalRepo;
    }

    public async Task Invoke(InsuranceClaim claim)
    {
        // Do some crazy logic from the data returned from each repository!

        // On failure, throw a general gate type exception that can be handled 
        // by middleware or a global error handler somewhere at the top of your stack.
        throw new GateFailureException("Insurance claim cannot be approved.")
    }
}
```

Each gate class will either succeed or fail. 

On failure, it will throw an exception that will be caught up the stack. In web applications, there is usually some global exception handler or middleware than can convert these into specific HTTP error responses, etc.

If we do need to use this logic in other places, as mentioned above, then we don't need to re-import all the dependencies required for this logic. We can just simply use the gate class as-is and allow the DI mechanism to plug in all the dependencies for us.

# Some Caveats

It's worth mentioning, that in some cases your use cases **and** your gate classes may need to call the same repository method. You don't want to be fetching that data twice (once in your gate class and once in your use case).

In this event, there are ways to fix it.

One is to build a **cached repository using the Decorator pattern**. 

Only the first attempt to fetch from the repository will actually fetch data. You might rig this up as a scoped dependency (in .NET Core) so the cached data will only be cached for the lifetime of the HTTP request.

Another way is to simply **allow the use case to inject the raw data into the gate class as a dependency**.

In any event, this pattern is very helpful in making your code much easier to test, use and maintain!

<hr />

## Keep In Touch

Don't forget to connect with me on [twitter](https://twitter.com/jamesmh_dev) or [LinkedIn](https://www.linkedin.com/in/jamesmhickey/)!

<div style="padding:0   20px; border-radius:6px; background-color: #efefef; margin-bottom:50px; margin-top:20px">
    <h1 class="margin-bottom:0"> Navigating Your Software Development Career
</h1>
An e-mail newsletter where I'll answer subscriber questions and offer advice around topics like:

✔ What are the general stages of a software developer?
✔ How do I know which stage I'm at? How do I get to the next stage?
✔ What is a tech leader and how do I become one?


<div class="text-center">
    <a href="http://eepurl.com/gdIV5X">
        <button class="btn btn-sign-up" style="margin-top:0;margin-bottom:0">Join The Community!</button>
    </a>
</div>
</div>

## You Might Also Enjoy

- [Where Do I Put My Business Rules And Validation?](https://builtwithdot.net/blog/where-do-i-put-my-business-rules-and-validation)(Guest post on [builtwithdot.net](https://builtwithdot.net))
- [.NET Core Dependency Injection: Everything You Ought To Know](https://www.blog.jamesmichaelhickey.com/NET-Core-Dependency-Injection/)
- [Keeping Your ADO Sql Connections Safe](https://www.blog.jamesmichaelhickey.com/keeping-ado-sql-connections-safe/)




