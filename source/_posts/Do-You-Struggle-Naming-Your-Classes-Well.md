---
title: Do You Struggle Naming Your Classes Well?
tags:
  - software design
date: 2019-06-05 12:10:06
---


Ever find yourself scratching your head while wondering what you should name that new class?

<!-- more -->

We all struggle with this!

In fact, [it's one of the two hardest parts of programming!](https://www.martinfowler.com/bliki/TwoHardThings.html)

Sometimes it's easy because you have a clear business concept in mind. 

Other times you might be creating classes that aren't necessarily tied to a business concept.

You might decide to split a class into several classes (for maintainability's sake). 

Or, you may need to use a design pattern.

🤦‍♂️

# Tips To The Rescue!

Writing code usually involved some specific scenario(s) that you are trying to solve. Here are some tips for tackling these kinds of business scenarios by naming your classes well:

1. Be as specific as possible!
2. Don't be afraid to be verbose at first. You can refactor later.
3. Try pretending that you are writing the outline for an article that explains your business process to a non-programmer. Start by naming things with the terms from this.

## Example

Here's what a sample of using tip #3 might look like:

> Once an order is submitted by the customer, we need to make sure that all the order's items are in stock. If some are not in stock, then we need to send them an email to let them know what items are on back-order.

> Next, we will pass the information to the shipping department for further handling.

We can use this description to create some classes from the nouns. Sometimes, depending on how you want to organize your code, it's very worthwhile to use the adjectives too!

- `Order` or `SubmittedOrder`
- `Customer` or `OrderCustomer`
- `OrderItems`
- `OrderItemsOnBackOrderEmail`
- `ShippingDepartmentOrderHandler`

Keep in mind that the nouns/terms are all applicable to the specific context (see [bounded contexts from DDD](https://www.martinfowler.com/bliki/BoundedContext.html)) you are trying to solve for.

The code we write might look like this at first pass:

```js
class OrderSubmittedHandler {

    public async handle(order: Order, customer: Customer) {
        const submitted: SubmittedOrder = order.submitOrder();

        if(submitted.hasItemsOnBackOrder()) {
            const mail = new OrderItemsOnBackOrder(submitted);
            mail.sendTo(customer);
            await this._mailer.send(mail);
        }

        const shipping = new ShippingDepartmentOrderHandler(submitted);
        await shipping.sendOrderInfo();
    }
    
}
```

Points of interest:

- Notice that we explicitly model the transition from an `Order` to a `SubmittedOrder`. Since a `SubmittedOrder` has different behaviours than an `Order`, this will keep your classes way smaller and manageable by following the [Single Responsibility Principle](https://www.weeklydevtips.com/episodes/049).

- We introduced cross-cutting objects like the `_mailer` variable. In this example, I already know that I am going to use dependency injection to supply the `Mailer`. But that can be something you decide to refactor later on.

- The entire scenario itself is captured by a meaningful noun as a new class!

   Of course, this can be refactored after-the-fact. But as a first pass, this technique can really help to solidify what to name things and how they ought to interact.

- Being verbose actually works well!

# Some More Guidelines

Here are some guidelines that might help you when dealing with specific kinds of classes that may or may not be obvious from a business scenario.

For example, sometimes you need to introduce a design pattern. What do you name your class then?

| Intent  |   Formula    | Examples             |
| --------|--------------|----------------------|
| Authorization | `Can{Entity}{Action}` | `CanAdminViewThisPage`, `CanManagerUpdateUserInfo`|
| Validation | `Is{Target}{State}{Test}` | `IsAddressUpdateAllowed`, `IsUserCreationValid` |
| Interfaces | `ICan{Action}` | `ICanSendMail`, `ICanBeValidated` |
| Concrete Business Concept | _"What is it?" (nouns + adjectives)_ | `Student`, `EmployeeUserProfile`, `ShippingAddress` |
| Use Cases | `{Action}{Target}` | `ApproveOrder`, `SendWelcomeEmail` |
| Design Pattern | `{Name}{Pattern}` | `IShippingAddressStrategy`, `HomeAddressStrategy`, `TemporaryAddressStrategy` |

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