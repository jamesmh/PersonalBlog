---
title: "C# Pattern Matching: Are The Official Docs Lying?"
tags:
  - "c#"
  - pattern matching
  - type checking
url: 652.html
id: 652
categories:
  - "C#"
date: 2018-06-11 16:32:45
---

During my day job, I had a case where I needed to use some pattern matching to do some type checking. If you don't know, pattern matching in C# allows you to test the type of an object and perform some additional "magic" at the same time. While having the chance to play around with this feature some questions arose from my usage.

<!--more-->

I want to bring you through some steps in typical pattern matching usage, and we'll ask some fun questions and test this feature to see how far we can bring it!

# Usage 1: If Statement

The common usage is by issuing type checking within an `if` statement.

```
object value = SomeFactory();

if(value is long asLong)
{
    // asLong is type <long>
}
else if(value is int asInt)
{
    // asInt is type <int>
}
else if(value is string str)
{
    // str is type <string>
}
```

Fair enough. Let's move onto the next common usage of pattern matching.

# Usage 2: Switch Statement

```
object value = SomeFactory();

switch (value)
{
    case long asLong:
        // asLong is type <long>
        break;
    case int asInt:
        // asInt is type <int>
        break;
    case string str:
        // str is type <string>
        break;
}
```

Looks good. Very convenient and useful.

But, this is where my brain has some questions. The syntax `[var] is [type] [newVar]` is interesting. It is not really a statement. It's an expression. But... it's also a statement. Why?

It's an expression because it evaluated to a `boolean`. But, it also makes an assignment to a new variable...

# Getting Fancy...

```
object value = SomeFactory();

bool isLong = value is long asLong;
bool isInt = value is int asInt;
bool isString = value is string str;

// ... etc.
```

This works. Each of these `boolean` values is set properly. In the [official docs](https://docs.microsoft.com/en-us/dotnet/csharp/pattern-matching#the-is-type-pattern-expression), the `is` operation is called the `is` **expression**. Fair enough.

This means we might be able to do this?

```
object value = SomeFactory();

bool isLong = value is long asLong;
bool isInt = value is int asInt;
bool isString = value is string str;

// Doesn't compile. "asLong" etc. are all in scope, but are "unassigned" (so Visual Studio tells me)
return isLong ? asLong
    : isInt ? asInt
    : isString ? str
    : new object();
```

Nope.

So the assignment to `asLong`, `asInt` and `str` seem to be scoped to the outer level - but the variables are just never set. That's what Visual Studio says. But, the [official C# docs](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/is#a-nametype--type-pattern-a) say:

> If exp is true and `is` is used with an if statement, `varname` is assigned and has local scope within the if statement only.

Alright. What if we did this?

```
object value = SomeFactory();

return value is long asLong ? asLong
    : value is int asInt ? asInt
    : value is string str ? str
    : new object();
```

That works and is a succinct way of expressing what we wanted to do.

# Getting Really Fancy

Did you notice though - what Visual Studio says and what the docs say is actually not perfectly in harmony? Visual Studio says that the variable (for example) `asLong` is in scope - it just hasn't been assigned. The docs say that when doing pattern matching in an `if` statement the variable is only in scope **within the `if` statement only**.

Let's play around with this.

```
if (value is long asLong)
    value = asLong;
else if (value is int asInt)
    value = asInt;
else if (value is string str)
    value = str;

asLong = 5; // This compiles! asLong is in scope!
asInt = 5; // Fails! asInt is not in scope at all!
str = ""; // Fails! str is not in scope at all!
```

Weird. `asLong` is actually in the function / outer scope. But not the other variables. Must be the way the compiler chooses to modify the code.

# Going The Extra Mile!

Ok... let's get super weird. Remember how in one of the code examples above we talked about the fact that the `is` operation is an expression?

```
bool isLong = value is long asLong;
bool isInt = value is int asInt;
bool isString = value is string str;

asLong = 5;
asInt = 5;
str = "";
```

Well, what do ya know! That works. Weird. Why?

Apparently, using the `is` expression in a **pure** `if` statement will declare the variable **above** the `if` statement. So what really ends up in your code after compiling is something like this:

```
if(value is long asLong)
{

}
else if(value is int asInt)
{

}

// Really becomes something like this by the compiler.

long asLong;
if(value is long)
{
    asLong = (long) value;
}
else if(value is int)
{
    int asInt = (int) value;
}

// NOT this (like the docs say).

if(value is long)
{
    long asLong = (long) value;
}
else if(value is int)
{
    int asInt = (int) value;
}
```

Neat.

In one of the code samples above - where we are assigning the result of the `is` expression to a boolean - the compiler must be treating each expression the same way it treats a pure `if` statement. Each variable is pushed up. Like this:

```
bool isLong = value is long asLong;
bool isInt = value is int asInt;
bool isString = value is string str;

// Must become...
long asLong;
bool isLong = value is long;

int asInt;
bool isInt = value is int;

string str;
bool isString = value is string;
```

# Are The Docs Wrong?

Again, the [official C# docs](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/is#a-nametype--type-pattern-a) say (with bold added by me):

> If exp is true and `is` is used with an if statement, `varname` is assigned and has local scope within the if statement **only**.

If we removed the last word of that statement, then it would be true. The variable doesn't have local scope `only` within the if statement. But as it is, it's technically false.

So - by being very technical and nit-picky - the docs are not totally 100% bang-on.

# Conclusion + Looking Ahead To C# 8

Does it matter? Not really. But, it's fun to play around with this stuff.

Using the ternary operator in conjunction with the `is` pattern matching is handy and very compact. But in C# version 8 we are expecting something that takes this to a whole new level!

Switch expressions are the "next level" and will look something like this:

```
object value = SomeFactory();

value switch
{
    long asLong => // Do something with it!,
    int asInt => // Do something else!,
    string str => // Do another thing!
};
```

Since this new usage of `switch` is an expression, you can return the result of that block of code. This should be great for things like the strategy pattern, factory patterns, etc.

# Appendix: Using IL Spy

In trying to push this feature as far as I could, I wrote some really weird code to figure out how the compiled code really works.

```
object value = 5;

if(value is int asInt)
{

}
else if(value is long asLong)
{
    asInt = 101;
}
else
{
    asInt = 102;
}

Console.Write(asInt);
```

Yes, that compiles. The Console writes `5`!

Changing the first line to `object value = 5L;` outputs `101`.

Changing it to `object value = 'g';` outputs `102`.

This confirms our conclusions from the article above.

But, by using IL Spy, we can see the **real code**. This is what is really outputted:

```
object value = 5;
int asInt;
object obj;
if ((obj = value) is int)
{
    asInt = (int)obj;
}
else if ((obj = value) is long)
{
    long num = (long)obj;
    asInt = 101;
}
else
{
    asInt = 102;
}
Console.Write((object)asInt);
```

Nice. If you follow closely (ya, it's hard to follow - I know...), this is exactly what my entire article was concluding.

What about this?

```
object value = 5;

bool isLong = value is long asLong;
bool isInt = value is int asInt;

Console.WriteLine(isLong == isInt);
```

It becomes:

```
object obj = 5;
int num;
object obj2;
if ((obj2 = obj) is long)
{
    long num3 = (long)obj2;
    num = 1;
}
else
{
    num = 0;
}
bool isLong = (byte)num != 0;
obj2 = obj;
int num2;
if (obj is int)
{
    int num4 = (int)obj2;
    num2 = 1;
}
else
{
    num2 = 0;
}
bool isInt = (byte)num2 != 0;
Console.WriteLine(isLong == isInt);
```

What? That's a mouthful. But, we can see that our conclusions about what **it seemed like the compiler is doing** are true.

Fun!

Hopefully, you learned something new! Let me know what you think!

_P.S. Here are some other articles you might enjoy!_

- [How I Made LINQ 6X Faster Using A Functional Optimization!](https://www.blog.jamesmichaelhickey.com/linq-6x-faster-using-functional-optimization/)
- [Refactoring Legacy Monoliths – Part 4: Refactoring Tools](https://www.blog.jamesmichaelhickey.com/refactoring-legacy-monoliths-part-4-refactoring-tools/)
- [Refactoring Legacy Monoliths Part 3: Refactoring Game Plan And Tips](https://www.blog.jamesmichaelhickey.com/refactoring-game-plan-and-tips/)
- [Deck The Halls With Strategy Pattern Implementations In C#: Basic To Advanced](https://www.blog.jamesmichaelhickey.com/strategy-pattern-implementations/)

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
