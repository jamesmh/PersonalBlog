---
title: 'Unhealthy Code: Primitive Overuse'
date: 2019-08-30 10:02:36
tags: refactoring, code quality, clean code
---

One of the classic "code smells" is called Primitive Overuse.

It's deceptively simple.

<!--more-->

----

_Note: This is an excerpt from my book [Refactoring TypeScript: Keeping Your Code Healthy.](https://leanpub.com/refactoringtypescript)_

[
![Refactoring TypeScript book](/img/refactoringts.png)
](https://leanpub.com/refactoringtypescript)

----

# Identification Of Primitive Overuse

Take this code, for example:

```typescript
const email: string = user.email;

if(email !== null && email !== "") {
    // Do something with the email.
}
```

Notice that we are handling the email's raw data?

Or, consider this:

```typescript
const firstname = user.firstname || "";
const lastname = user.lastname || "";
const fullName: string = firstname + " " + lastname;
```

Notice all that extra checking around making sure the user's names are not `null`? You've seen code like this, no doubt.

## What's Wrong Here?

What's wrong with this code? There are a few things to think about:

- That logic is not sharable and therefore will be duplicated all over the place

- In more complex scenarios, it's hard to see what the underlying business concept represents (which leads to code that's hard to understand)

- If there is an underlying business concept, it's implicit, not explicit

## Business Concepts By Chance

The business concept in the code example above is something like a user's _display name_ or _full name_.

However, that concept only exists temporarily in a variable _that just happened to be named correctly._ Will it be named the same thing in other places? If you have other developers on your team - **probably not**. 

We have code that's potentially hard to grasp from a business perspective, hard to understand in complex scenarios and is not sharable to other places in your application.

How can we deal with this?

# Deceptive Booleans

Primitive types should be the building blocks out of which we create more useful business-oriented concepts/abstractions in our code.

This helps each specific business concept to have all of its logic in one place (which means we can share it and reason about it much easier), implement more robust error handling, reduce bugs, etc.

I want to look at the most common cause of primitive overuse that I've experienced. I see it _all the time_.

## Scenario

Imagine we're working on a web application that helps clients to sell their used items online.

We've been asked to add some extra rules around the part of our system that authenticates users.

Right now, the system only checks if a user was successfully authenticated.

```typescript
const isAuthenticated: boolean = await userIsAuthenticated(username, password);

if(isAuthenticated) {
    redirectToUserDashboard();
} else {
    returnErrorOnLoginPage("Credentials are not valid.");
}
```

## New Business Rules

Our company now wants us to check if users are active. Inactive users will not be able to log in.

Many developers will do something like this:

```typescript
const user: User = await userIsAuthenticated(username, password);
const isAuthenticated: boolean = user !== null;

if(isAuthenticated) {
    if(user.isActive) {
        redirectToUserDashboard();
    } else {
        returnErrorOnLoginPage("User is not active.");
    }
} else {
    returnErrorOnLoginPage("Credentials are not valid.");
}
```

Oh no. We've introduced code smells that we know are going to cause maintainability issues!

We've got some null checks and nested conditions in there now (which are both signs of unhealthy code that are addressed in the [Refactoring TypeScript](https://leanpub.com/refactoringtypescript) book.)

So, let's refactor that first by applying (a) the special case pattern and (b) guard clauses (both of these techniques are explained at length in [the book](https://leanpub.com/refactoringtypescript) too.)

```typescript
// This will now always return a User, but it may be a special case type
// of User that will return false for "user.isAuthenticated()", etc.
const user: User = await userIsAuthenticated(username, password);

// We've created guard clauses here.
if(!user.isAuthenticated()) {
    returnErrorOnLoginPage("Credentials are not valid.");
}

if(!user.isActive()) {
    returnErrorOnLoginPage("User is not active.");
}

redirectToUserDashboard();
```

Much better.

## More Rules...

Now that your managers have seen how fast you were able to add that new business rule, they have a few more they need.

1. If the user's session already exists, then send the user to a special home page.

2. If the user has locked their account due to too many login attempts, send them to a special page.

3. If this is a user's first login, then send them to a special welcome page.

**Yikes!**

> If you've been in the industry for a few years, then you know how common this is!

At first glance, we might do something naïve:

```typescript
// This will now always return a User, but it may be a special case type
// of User that will return false for "user.isAuthenticated()", etc.
const user: User = await userIsAuthenticated(username, password);

// We've created guard clauses here.
if(!user.isAuthenticated()) {
    returnErrorOnLoginPage("Credentials are not valid.");
}

if(!user.isActive()) {
    returnErrorOnLoginPage("User is not active.");
}

if(user.alreadyHadSession()) {
    redirectToHomePage();
}

if(user.isLockedOut()) {
    redirectToUserLockedOutPage();
}

if(user.isFirstLogin()) {
    redirectToWelcomePage();
}

redirectToUserDashboard();
```

Notice that because we introduced guard clauses, it's much easier to add new logic here? That's one of the awesome benefits of making your code high-quality - it leads to future changes being _much_ easier to change and add new logic to.

But, in this case, there's an issue. Can you spot it?

**Our `User` class is becoming a dumping ground for all our authentication logic.**

## Is It Really That Bad?

Is it that bad? _Yep._

Think about it: what other places in your app will need this data? Nowhere - it's all authentication logic.

One refactoring would be to create a new class called `AuthenticatedUser` and put only authentication-related logic in that class.

This would follow the Single Responsibility Principle.

But, there's a much simpler fix we could make for this specific scenario.

## Just Use Enums

Any time I see this pattern (the result of a method is a boolean or is an object that has booleans which are checked/tested immediately), it's a much better practice to replace the booleans with an enum.

From our last code snippet above, let's change the method `userIsAuthenticated` to something that more accurately describes what we are trying to do: `tryAuthenticateUser`.

And, instead of returning either a `boolean` or a `User` - we'll send back an enum that tells us exactly what the results were (since that's all we are interested in knowing).

```typescript
enum AuthenticationResult {
    InvalidCredentials,
    UserIsNotActive,
    HasExistingSession,
    IsLockedOut,
    IsFirstLogin,
    Successful
}
```

There's our new enum that will specify all the possible results from attempting to authenticate a user.

Next, we'll use that enum:

```typescript
const result: AuthenticationResult = await tryAuthenticateUser(username, password);

if(result === AuthenticationResult.InvalidCredentials) {
    returnErrorOnLoginPage("Credentials are not valid.");
}

if(result === AuthenticationResult.UserIsNotActive) {
    returnErrorOnLoginPage("User is not active.");
}

if(result === AuthenticationResult.HasExistingSession) {
    redirectToHomePage();
}

if(result === AuthenticationResult.IsLockedOut) {
    redirectToUserLockedOutPage();
}

if(result === AuthenticationResult.IsFirstLogin) {
    redirectToWelcomePage();
}

redirectToUserDashboard();
```

Notice how much more readable that is? And, we aren't polluting our `User` class anymore with a bunch of extra data that is unnecessary!

We are returning _one value_. This is a great way to simplify your code. 

This is one of my favorite refactorings! I hope you will find it useful too.

## Bonus: Strategy Pattern

Whenever I use this refactoring, I know automatically that the strategy pattern may help us some more.

Imagine the code above had _lots_ more business rules and paths.

We can further simplify it by using a form of the strategy pattern:

```typescript
const strategies: any = [];

strategies[AuthenticationResult.InvalidCredentials] = 
    () => returnErrorOnLoginPage("Credentials are not valid.");
strategies[AuthenticationResult.UserIsNotActive] = 
    () => returnErrorOnLoginPage("User is not active.");
strategies[AuthenticationResult.HasExistingSession] = 
    () => redirectToHomePage();
strategies[AuthenticationResult.IsLockedOut] = 
    () => redirectToUserLockedOutPage();
strategies[AuthenticationResult.IsFirstLogin] = 
    () => redirectToWelcomePage();
strategies[AuthenticationResult.Successful] = 
    () => redirectToUserDashboard();

strategies[result]();
```

# How To Keep Your Code Healthy

This post was an excerpt from [Refactoring TypeScript](https://leanpub.com/refactoringtypescript) which is designed as an approachable and practical tool to help developers get better at building quality software. 

[![Refactoring TypeScript book](/img/refactoringts.png)
](https://leanpub.com/refactoringtypescript)

## Keep In Touch

Don't forget to connect with me on [twitter](https://twitter.com/jamesmh_dev) or [LinkedIn](https://www.linkedin.com/in/jamesmhickey/)!

<div style="padding:0 20px; border-radius:6px; background-color: #efefef; margin-bottom:50px; margin-top:20px">
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

- [Why Should You Refactor Your Code?](https://www.blog.jamesmichaelhickey.com/why-should-you-refactor-your-code/)
- [Refactoring Legacy Monoliths - Part 1: First Steps](https://www.blog.jamesmichaelhickey.com/refactoring-legacy-monoliths-first-steps/)
- [Where Do I Put My Business Rules And Validation?](https://builtwithdot.net/blog/where-do-i-put-my-business-rules-and-validation)

