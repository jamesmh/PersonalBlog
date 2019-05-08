---
title: 'Practical Coding Patterns For Boss Developers #1: Special Case'
date: 2019-05-08 08:00:00
tags: 
    - design patterns
    - refactoring
---

Design patterns are necessary (in my opinion) for you to start gaining an advanced understanding and ability to design and refactor software.

These patterns also give developers a common language to speak about certain code structures.

_E.g. If you are facing a certain problem, and I say "Try the strategy pattern..." then I don't have to spend an hour explaining what you should do._

You can go look it up or, if you already know the pattern, go implement it!

# Not Your Typical Design Patterns

Everyone is talking about the typical design patterns found in the gang of four book [Design Patterns](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612/ref=sr_1_1?keywords=design+patterns&link_code=qs&qid=1557247400&s=gateway&sr=8-1):

- Strategy
- Builder
- Factory
- Adapter
- etc.

But, there are **way more** software design patterns not found in this book which I've found super helpful.

Some of them I've learned from [Martin Fowler](https://www.martinfowler.com), others from [Domain Driven Design](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215/ref=sr_1_1?keywords=domain+driven+design&link_code=qs&qid=1557247688&s=gateway&sr=8-1), and other sources.

I figured I'd start to catalogue some of these by sharing them!

As a general rule, I'm going to use TypeScript for my code samples.

# A Close Relative...

The pattern I want to start with is a close relative to the null object pattern that I've tweeted about before:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/hashtag/csharp?src=hash&amp;ref_src=twsrc%5Etfw">#csharp</a> <a href="https://twitter.com/hashtag/dotnet?src=hash&amp;ref_src=twsrc%5Etfw">#dotnet</a> <br><br>C# tip 🔥 - a form of the null object pattern 👇 <a href="https://t.co/fLpO5ibLhO">pic.twitter.com/fLpO5ibLhO</a></p>&mdash; James Hickey 🇨🇦👨‍💻 (@jamesmh_dev) <a href="https://twitter.com/jamesmh_dev/status/1067605234653061120?ref_src=twsrc%5Etfw">November 28, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


The null object pattern is a way of avoiding issues around `null` states (like all the extra `null` checking you need to do 😒).

You create a specialized version of a class to represent it in a "null" state, which exposes the same API or interface as the base object.

In other words, it's kinda like a stub.

Here's an example off the top of my head, in TypeScript:

```ts
// Concrete Order
class Order {
    private _items: any[];

    constructor(items: any[]) {
        this._items = items;
    }

    public placeOrder() {
        // API call or whatever...
    }
}

// Null "version" of the Order
class NullOrder extends Order {
    constructor() {
        super([]);
    }

    public placeOrder() {
        // We just do nothing!
        // No errors are thrown!
    }
}

// Usage:
const orders: Order[] = [
    new Order(['fancy pants', 't-shirt']), new NullOrder()
];

for (const order of orders) {
    // This won't throw on nulls since we've
    // used the null object pattern.
    order.placeOrder();
}
```

# An Example Scenario

Imagine we had a scheduled background process that fetches multiple orders and tries to place them. 

Like Amazon, we might have a complex process around placing orders that isn't as linear as we might think it would be.

We might want a buffer period between the time when you click "place order" and when the order is **really** placed. This will make cancelling an order an easy process (it avoids having to remove credit card charges, etc.)

_Note: I might write about this pattern later._ 😉

In this scenario, we might be trying to process orders that have already been changed:

- Cancelled order
- Payment declined
- The payment gateway is not responding so we need to wait...
- etc.

The null object pattern can help with this kind of scenario.

**But even better, when you have multiple versions of these kinds of "special" cases, the special case pattern is here to save the day!**

# Special Case Pattern

The special case pattern is essentially the same in implementation, but instead of modelling specific "null" states, we can use the same technique to model _any_ special or non-typical cases.

Using the code example above, instead of having "null" versions of our `Order`, by using the special case pattern we can implement more semantic and targeted  variants of our `Order` class:

```ts
class IncompleteOrder extends Order {
    constructor() {
        super([]);
    }

    public placeOrder() {
        // Do nothing...
    }
}

class CorruptedOrder extends Order {
    constructor() {
        super([]);
    }

    public placeOrder() {
        // Try to fix the corruption?
    }
}

class OrderOnFraudulentAccount extends Order {
    constructor() {
        super([]);
    }

    public placeOrder() {
        // Notify the fraud dept.?
    }
}
```

As you can see, this pattern helps us to be very specific in how we model special cases in our code.

# Benefits

Some benefits are:

- Avoiding `null` exception issues
- Having classes with more targeted single responsibilities
- Ability to handle special cases without our `Order` class blowing up with tons of logic
- The semantics of the class names makes our code much more understandable to read
- Introducing new cases involves creating new classes vs. changing existing classes ([see open-closed principle](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle))

# Refactoring To The Special Case Pattern

So, when should you use the pattern?

### Constructor

You might consider this pattern whenever you see this type of logic in a class' constructor:

```ts
constructor() {
    if(this.fraudWasDetected()) {
        this._fraudDetected = true;
    } else {
        this.fraudWasDetected = false;
    }
}
```

_Note: The refactoring for this will begin in the [Oops, Too Many Responsibilities](#oops-too-many-responsibilities) section below._

### Outside "Asking"

When you see something like the following, then you may want to consider the special case pattern:

```ts
const order = getOrderFromDB();

if(order.fraudWasDetected()) {
    order.doFraudDetectedStuff();
} else if(!order.hasItems()) {
    order.placeOrder();
} 
// ... and more ...
```

Focusing on this example, why is this potentially a "code smell"?

At face value, this type of logic should be "baked into" the Order class(es).

Whoever is using the order shouldn't have to know about this logic. This is all order specific logic. It shouldn't be "asking" the `Order` for details and then deciding how to use the `Order`.

For more, see the [tell don't ask principle](https://www.martinfowler.com/bliki/TellDontAsk.html) - which, most times, does indicate that your logic might be better suited inside the object you are using.

The first fix then is to move this logic to the _inside_ of the `Order` class:

```ts
class Order {

    public placeOrder() {
        if(this._fraudWasDetected()) {
            this._doFraudDetectedStuff();
        } else if(!this._hasItems()) {
            this._placeOrder();
        } 
    }
}
```

### Oops, Too Many Responsibilities!

But, now we run into some issues: we are dealing with different responsibilities (placing orders, fraud detection, item corruption, etc.) in one class! 😓

_Note: What follows can be applied to the constructor refactor too._

**Special case pattern to the rescue!**

```ts
// Note: I'm just highlighting the main parts, this won't compile 😋
class CorruptedOrder extends Order {

    public placeOrder() {
        this._fixCorruptedItems();
        super.placeOrder();
    }
}

class OrderOnFraudulentAccount extends Order {

    public placeOrder() {
        this._notifyFraudDepartment();
    }
}

class IncompleteOrder extends Order {

    public placeOrder() {
        // Do nothing...
    }
}

```

### Combining Patterns

Great! But how can we instantiate these different classes?

The beauty of design patterns is that they usually end up working together. In this case, we could use the factory pattern:

```ts
class OrderFactory {

    public static makeOrder(accountId: number, items: any[]): Order {
        if (this._fraudWasDetected(accountId)) {
            return new OrderOnFraudulentAccount();
        } else if (items === null || items.length === 0) {
            return new EmptyOrder();
        }
        // and so on....
    }
}

```

_Note: This is a pretty clear example of when you would really want to use the factory pattern - which I do find can be easily over-explained. Hopefully, this helps you to see why you would want to use a factory in the first place._

We have split our `Order` class into a few more special classes that can each handle one special case.

Even if the logic for, let's say, processing a suspected fraudulent account/order is very complex, we have isolated that complexity into a targeted and specialized class.

As far as the consumers of the `Order` class(es) go - they have no idea what's going on under the covers! They can simply call `order.placeOrder()` and let each class handle its own special case.

# Resources

- [Special case pattern by Martin Fowler](https://www.martinfowler.com/eaaCatalog/specialCase.html)
- [Null object design pattern](https://sourcemaking.com/design_patterns/null_object)

# Thoughts?

Have you ever encountered the special case pattern? Or perhaps any of the others I've mentioned?

# Keep In Touch

Don't forget to connect with me on:

- [Twitter](https://twitter.com/jamesmh_dev)
- [LinkedIn](https://www.linkedin.com/in/jamesmhickey/)

You can also find me at my web site [www.jamesmichaelhickey.com](https://www.jamesmichaelhickey.com).

# Navigating Your Software Development Career Newsletter

An e-mail newsletter that will help you level-up in your career as a software developer! Ever wonder:

✔ What are the general stages of a software developer?
✔ How do I know which stage I'm at? How do I get to the next stage?
✔ What is a tech leader and how do I become one?
✔ Is there someone willing to walk with me and answer my questions?

Sound interesting? [Join the community!](https://eepurl.com/gdIV5X)
