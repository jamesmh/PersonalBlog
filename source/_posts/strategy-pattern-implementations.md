---
title: "6 Ways To Implement The Strategy Pattern In C# (Basic To Advanced)"
tags:
  - "c#"
  - "c# design patterns"
  - "c# patterns"
  - functional programming
  - "functional programming c#"
  - object oriented design
  - object oriented design patterns
  - strategy
  - strategy pattern
url: 469.html
id: 469
categories:
  - "C#"
  - Functional Programming
  - Object Oriented Programming
  - Software Architecture And Design
date: 2017-12-09 04:10:43
---

Let's look at some different ways you could implement the strategy pattern in C#. First, I'd like to briefly mention why we care about design patterns and where the strategy pattern fits in.

_Note: [This is day #9 of the First C# Advent Calendar](https://crosscuttingconcerns.com/The-First-C-Advent-Calendar) @ [https://crosscuttingconcerns.com](https://crosscuttingconcerns.com)_

<!--more-->

# Why Should I Know The Strategy Pattern?

Understanding design patterns is a vital skill to possess as a software developer and/or software architect. If you don't, then you end up:

- Wasting time while solving problems which have already been addressed by the community
- Creating code that is potentially difficult to maintain
- Writing code that is potentially difficult to reason about
- Building software components and pieces that are potentially difficult to extend

The strategy pattern is a way of approaching problems where you have different paths of logic that are available and based on some condition(s) you need to choose one of those paths. In other words, you have too many `if` or `switch` cases and need a cleaner more extensible alternative. 😉

# The Flow Of The Strategy Pattern

The strategy pattern follows a basic flow:

1. An entry point accepts a choice that was made (by a user, system, etc.)
2. Based on this selection, one-out-of-many algorithms or paths of logic is selected to execute.
3. Execute the selected algorithm.

# Practical Uses Of The Strategy Pattern

A practical example that some (most?) web developers have faced is building business-oriented reporting systems. A reporting system in a web app is a good use-case for the strategy pattern:

- Users can choose a certain report to execute from a list of available reports
- Users can export the report in different formats
- Etc.

Imagine a system with 100 types of reports to choose. You could create 100 links on your web page. You could use 100 `if` statements. 100 `switch` cases. But what if users can now choose a selection of reports that get automated into an email every day? Now you can't use links. And those `if` statements get pretty messy and unmaintainable - especially as new requirements cause nested `if` statements. **There is a better way - the strategy pattern.**

# Basic Implementation

Let's look a very basic implementation. I'm going to use a very "stupid simple" example of a console app calculator having 4 operations (always two numbers):

- Add
- Subtract
- Multiply
- Divide

This is in an attempt to avoid distractions in trying to grasp this pattern, which can be applied to more complicated business problems.

Here's the basic example:

```
// We have 4 methods that perform a calculation using 2 integers.
// These are our available algorithms to select from.
private static int Add(int a, int b) => a + b;
private static int Minus(int a, int b) => a - b;
private static int Multiply(int a, int b) => a * b;
private static int Divide(int a, int b) => a / b;
```

Ignoring user input and output, in the same class you would have the "strategy":

```
switch (userChoice)
{
    case "+":
        result = Add(numOne, numTwo);
        break;
    case "-":
        result = Minus(numOne, numTwo);
        break;
    case "*":
        result = Multiply(numOne, numTwo);
        break;
    case "/":
        result = Divide(numOne, numTwo);
        break;
}
```

This implementation is:

- Quick
- Easy
- Simple

But with more algorithms - let's say - dozens more, the file and/or class that holds all these methods and logic will become bloated very fast. And that switch statement starts to become a monster...

## When To Use?

This implementation is:

- A good step for those starting with the pattern
- Simple for using a very small number of choices

# Basic With Switch Statement Removal

This next step in the progression is to clean up the `switch` statement. It's too verbose - even for places with a small number of choices. The `switch` becomes:

```
Dictionary<string, Func<int, int, int>> strategies = new Dictionary<string, Func<int, int, int>>() {
    { "+", Add }, // Holding a reference to the method Add()
    { "-", Minus },
    { "*", Multiply },
    { "/", Divide }
};

Func<int, int, int> selectedStrategy = strategies[userChoice]; // userChoice is either "+", "-", "*" or "/"
int result = selectedStrategy(numOne, NumTwo);
```

By using a Dictionary, instead of explicitly invoking the method we want, we store a lookup of a `key` and `value` - the `value` being a reference to the method we want to Invoke for that `key`.

## When To Use?

This way forces you to create other methods to handle each case, instead of inlining more logic in the current scope. You might want to use this implementation if:

- You have some `if` or `switch` statements that you want to be less verbose
- You have a "handful" of algorithms available to choose

The dictionary definition may be too verbose for some. Let's keep reading...

# Using Interfaces

The next step is to clean up the list of methods and generalize them. What if we had another class somewhere else in our system that needed to be available to our choices of algorithms? Not to say this is a good design practice or not, but stuff happens in the real world.

Using an interface to abstract the implementation away is what we'll do here. This is usually considered the "classic" implementation of the strategy pattern.

**First**, we create a new interface to abstract our math operations:

```
public interface IMathOperator
{
     int Operation(int a, int b);
}
```

**Next**, we implement the interface for each algorithm (one example here):

```
public class MathAdd : IMathOperator
{
    public int Operation(int a, int b)
    {
        return a + b;
    }
}
```

**Finally**, we can just use the interface type `IMathOperator` in our calling code:

```
Dictionary<string, IMathOperator> strategies = new Dictionary<string, IMathOperator>() {
    { "+", new MathAdd() },
    { "-", new MathSubtract() },
    { "*", new MathMultiply() },
    { "/", new MathDivide() }
};

IMathOperator selectedStrategy = strategies[userChoice];
int result = selectedStrategy.Operation(numOne, numTwo);
```

Now we are able to separate each algorithm into its own class/file. This allows us to look at our file system and get a better overview of what our system can do. And, it's breaking up our code more which leads to having less visual noise per file.

## When To Use?

As this is the most common pattern implementation, you might use this version because:

- It helps when working with a team of developers (either they already understand the pattern or when researching the pattern this is probably what they will come across)
- Number of algorithms are expanding
- You want to have each algorithm in its own file/class (maybe each algorithm is really complex, etc.)

Again, you can get a better overview of your system since each algorithm is in its own file. Using interfaces you also allow some flexibility for existing classes to implement this abstraction.

# Expensive Object Instantiation?

A problem with the method above is that we are instantiating all the classes that we are choosing from. What if instantiating these objects is expensive? Even if not, using the following method is a quick fix that can improve performance - especially if we have a lot of implementations to choose from.

```
// Each key now returns a function that will instantiate an object when executed.
Dictionary<string, Func<IMathOperator>> strategies = new Dictionary<string, Func<IMathOperator>>() {
    { "+", () => new MathAdd() },
    { "-", () => new MathSubtract() },
    { "*", () => new MathMultiply() },
    { "/", () => new MathDivide() }
};

IMathOperator selectedStrategy = strategies[userChoice](); // Invoke the Func to instantiate new object
```

## When To Use?

You are using the Dictionary technique and...

- You are using expensive objects (e.g. objects that are part of an inheritance hierarchy, objects that perform some expensive operation when instantiated, objects that hold a lot of data, etc.)
- There are many algorithms available (and don't want to allocate all that extra memory)

# Dynamic Instantiation

You may have noticed that the strategy pattern is basically the same as the factory pattern. The factory pattern in OOP generally follows the flow:

1. Makes a decision to choose a concrete type of a higher level abstraction (interface, abstract class, etc.)
2. Instantiates it
3. Returns the new object.

The strategy pattern is similar in that it may do everything in the flow above, but adds one more step:

- Invoke a concrete implementation of a certain abstract method (i.e. Invoke an interface's method from the concrete object).

That leads us to the idea that we can use advanced techniques used in the factory pattern to enhance our strategy pattern implementation. One of these is to instantiate objects dynamically based on the name of a class/type. In our C# implementation, we can replace the Dictionary with this, assuming that our user input is now "Add" instead of "+", etc.:

```
IMathOperator selectedStrategy = GetMathOperatorFromFullClassPath($"Strategy.InterfacesExample.Math{userChoice}");
```

Factory method:

```
 private static IMathOperator GetMathOperatorFromFullClassPath(string fullClassPath) =>
         // The true's mean throw exception and ignoreCase.
         Activator.CreateInstance(Type.GetType(fullClassPath, true, true)) as IMathOperator;
```

If we know the namespace of where our concrete implementations are, then we can dynamically instantiate the one we want based on the user's input, or a database result, etc.

## When To Use?

- You have many concrete types
- You want to generalize the factory logic to be more concise
- Quick to add new implementations since you don't need to go digging in the factory again (e.g. just add a new class and choice for the user corresponding to that class' name)

Cons are:

- You introduce the possibility of more runtime errors
- Need to consider error handling more closely
- Code may not be as intuitive for newcomers to your project
- Future refactoring becomes harder since you cannot find references to places your classes may be used

# Last One: Advanced Factory

One last step! Let's introduce a more advanced/dynamic way of using reflection to dynamically "discover" which concrete implementation we may need inside our strategy pattern. But using reflection in C# we can find all the types in the current Assembly (i.e. project or library) that implement the interface we are looking for (IMathOperator). Here's what the strategy piece looks like:

```
// I.e. I want a "IMathOperator" object with the class name "Math[Add]" or "Math[Subtract]", etc.
IMathOperator selectedStrategy = Instantiate<IMathOperator>($"Math{userChoice}");
int result = selectedStrategy.Operation(numOne, numTwo);
```

Refactoring the factory function into a more flexible/reusable piece, we get:

```
private static T Instantiate<T>(string className)
{
    Type typeImplemented = typeof(T);
    Type selectedType = Assembly.GetExecutingAssembly() // i.e. our project/library
            .GetTypes()
            .First(t => typeImplemented.IsAssignableFrom(t) && t.Name == className);

    return (T) Activator.CreateInstance(selectedType);
}
```

## When To Use?

- You want a reusable factory function that you can use everywhere
- You have a large number of concrete strategies available
- You are using the strategy pattern in other places and want a more generalized mechanism to handle them all

Cons

- Potential for runtime errors
- Potential for naming conflicts between classes (i.e. two classes names "MathAdd" in different namespaces might cause runtime errors if you aren't careful with what types they implement)
- Not obvious how this piece may work to your project's newcomers or even intermediate developers

# Conclusion

The strategy pattern is helpful in eliminating a growing number of `if` and `switch` cases. By using interfaces or, in simpler cases, plain functions (i.e. static methods), we can introduce a more flexible, maintainable and predictable way of dealing with choosing specific algorithms or paths of logic.

My preference is to use a Dictionary to map keys of possible incoming values ("input") to the corresponding interfaces or methods to be selected and executed.

Doing this using Reflection can offer some conveniences and perhaps more flexibility, but the issue of losing track of all references to where your specific implementations are being used (since the concrete implementations are dynamically instantiated) is a pretty huge consideration.

<hr />

# Keep In Touch

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

- [What I've Learned So Far Building Coravel (Open Source .NET Core Tooling)](https://www.blog.jamesmichaelhickey.com/What-I-ve-Learned-So-Far-Building-Coravel-Open-Source-NET-Core-Tooling/)
- [Fluent APIs Make Developers Love Using Your .NET Libraries (Guest post on BuiltWithDot.Net)](https://builtwithdot.net/blog/fluent-apis-make-developers-love-using-your-net-libraries)
- [.NET Core Dependency Injection: Everything You Ought To Know](https://www.blog.jamesmichaelhickey.com/NET-Core-Dependency-Injection/)
