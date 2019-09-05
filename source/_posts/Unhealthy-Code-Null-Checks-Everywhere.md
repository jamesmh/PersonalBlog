---
title: 'Unhealthy Code: Null Checks Everywhere!'
date: 2019-09-04 23:08:14
tags: refactoring, code quality, clean code
---


# Identifying The Problem

## Billion Dollar Mistake

Did you know that the inventor of the concept of "null" has called this his ["Billion Dollar Mistake!"](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/)

As simple as it seems, once you get into larger projects and codebases you'll inevitably find some code that goes "off the deep end" in its use of nulls.

<!--more-->

----

_Note: This is an excerpt from my book [Refactoring TypeScript: Keeping Your Code Healthy.](https://leanpub.com/refactoringtypescript)_

[
![Refactoring TypeScript book](/img/refactoringts.png)
](https://leanpub.com/refactoringtypescript)

----

Sometimes, we desire to simply make a property of an object optional:

```typescript
class Product{
  public id: number;
  public title: string;
  public description: string;
}
```

In TypeScript, a `string` property can be assigned the value `null`. 

But... so can a `number` property!

```typescript
const chocolate: Product = new Product();
chocolate.id = null;
chocolate.description = null;
```

_Hmmm...._

## Another Example

That doesn't look so bad at first glance.

But, it can lead to the possibility of doing something like this:

```typescript
const chocolate: Product = new Product(null, null, null);
```

What's wrong with that? Well, it allows your code (in this case, the `Product` class) to get into an inconsistent state.

Does it ever make sense to have a `Product` in your system that has no `id`? Probably not. 

Ideally, as soon as you create your `Product` it should have an `id`.

So... what happens in other places that have to deal with logic around dealing with Products?

Here's the sad truth:

```typescript
let title: string;

if(product != null) {
    if(product.id != null) {
        if(product.title != null) {
            title = product.title;
        } else {
            title = "N/A";
        }
    } else {
        title = "N/A"
    }
} else {
    title = "N/A"
}
```

_Is that even real code someone would write?_

**Yes.**

Let's look at why this code is unhealthy and considered a "code smell" before we look at some techniques to fix it.

## Is It That Bad?

This code is hard to read and understand. Therefore, it's very prone to bugs when changed.

I think we can agree that having code like this scattered in your app is not ideal. Especially when this kind of code is inside the important and critical parts of your application!

-----

## A Side-Note About Non-Nullable Types In TypeScript

As a relevant side note, someone might raise the fact that TypeScript supports [non-nullable types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#nullable-types).

This allows you to add a special flag to your compilation options and will prevent, by default, any variables to allow `null` as a value.

A few points about this argument:

- Most of us are dealing with existing codebases that would take **tons** of work and time to fix these compilation errors.

- Without testing the code well, and carefully avoiding assumptions, we could **still** potentially cause run-time errors by these changes.

- This article (taken from [my book](https://leanpub.com/refactoringtypescript)) teaches you about solutions that can be applied to other languages - which may not have this option available.

-----

Either way, it's always safer to apply smaller more targeted improvements to our code. Again, this allows us to make sure the system still behaves the same and avoids introducing a large amount of risk when making these improvements.

# One Solution: Null Object Pattern

## Empty Collections

Imagine you work for a company that writes software for dealing with legal cases. 

As you are working on a feature, you discover some code:

```typescript
const legalCases: LegalCase[] = await fetchCasesFromAPI();
for (const legalCase of legalCases) {
    if(legalCase.documents != null) {
        uploadDocuments(legalCase.documents);
    }
}
```

Remember that we should be wary of null checks? What if some other part of the code forgot to check for a `null` array?

The Null Object Pattern can help: you create an object that represents an "empty" or `null` object.

### Fixing It Up

Let's look at the `fetchCasesFromAPI()` method. We'll apply a version of this pattern that's a very common practice in JavaScript and TypeScript when dealing with arrays:

```typescript
const fetchCasesFromAPI = async function() {
    const legalCases: LegalCase[] = await $http.get('legal-cases/');

    for (const legalCase of legalCases) {
        // Null Object Pattern
        legalCase.documents = legalCase.documents || [];
    }
    return legalCases;
}
```

Instead of leaving empty arrays/collections as `null`, we are assigning it an actual empty array.

Now, no one else will need to make a null check!

But... what about the entire legal case collection itself? What if the API returns `null`?

```typescript
const fetchCasesFromAPI = async function() {
    const legalCasesFromAPI: LegalCase[] = await $http.get('legal-cases/');
    // Null Object Pattern
    const legalCases = legalCasesFromAPI || [];

    for (const case of legalCases) {
        // Null Object Pattern
        case.documents = case.documents || [];
    }
    return legalCases;
}
```

Cool!

Now we've made sure that everyone who uses this method does not need to be worried about checking for nulls.

## Take 2

Other languages like C#, Java, etc. won't allow you to assign a mere empty array to a collection due to rules around strong typing (i.e. `[]`).

In those cases, you can use something like this version of the Null Object Pattern:

```typescript
class EmptyArray<T> {
    static create() {
        return new Array<T>()
    }
}

// Use it like this:
const myEmptyArray: string[] = EmptyArray<string>.create();
```

----
## Side-Note: Singleton

Sometimes, depending on the context, this is done using a [singleton](https://sourcemaking.com/design_patterns/singleton):

```typescript

class EmptyArray<T> {
    private static _instance = new Array<T>();

    static create() {
        return EmptyArray<T>._instance;
    }
}
```
----

## What About Objects?

Imagine that you are working on a video game. In it, some levels might have a boss.

When checking if the current level has a boss, you might see something like this:

```typescript
if(currentLevel.boss != null) {
    currentLevel.boss.fight(player);
}
```

We might find other places that do this null check:

```typescript
if(currentLevel.boss != null) {
    currentLevel.completed = currentLevel.boss.isDead();
}
```

If we introduce a null object, then we can simply remove all these null checks.

First, we need an interface to represent our `Boss`:

```typescript
interface IBoss {
    fight(player: Player);
    isDead();
}
```

Then, we can create our concrete boss class:

```typescript
class Boss implements IBoss {
    fight(player: Player) {
        // Do some logic and return a bool.
    }
    
    isDead() {
        // Return whether boss is dead depending on how the fight went.
    }
}
```

Next, we'll create an implementation of the `IBoss` interface that represents a "null" `Boss`:

```typescript
class NullBoss implements IBoss {
    fight(player: Player) {
        // Player always wins.
    }
    isDead() {
        return true;
    }
}
```

The `NullBoss` will automatically allow the player to "win", and we can remove all our null checks!

In the following code example, if the boss is an instance of `NullBoss` or `Boss` there are no extra checks to be made.

```typescript
currentLevel.boss.fight(player);
currentLevel.completed = currentLevel.boss.isDead();
```

_Note: This section in the book contains more techniques to attack this code smell!_

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
- [Unhealthy Code: Primitive Overuse](https://www.blog.jamesmichaelhickey.com/Unhealthy-Code-Primitive-Overuse/)

