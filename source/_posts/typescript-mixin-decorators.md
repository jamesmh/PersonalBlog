---
title: Using TypeScript Decorators To Clean-Up Mixins
tags:
  - decorators
  - javascript
  - mixins
  - typescript
url: 109.html
id: 109
categories:
  - TypeScript
date: 2017-02-21 15:56:46
---

Lately (at the time of publishing this article) I've been migrating a JavaScript framework I built over to TypeScript. The core UI components are built using [mixins](https://en.wikipedia.org/wiki/Mixin#In_JavaScript "mixins"). One issue I ran into is trying to clean up code that builds creates objects by combining mixins. By using TypeScript decorators I was able to clean this up to my liking. :)

<!--more-->

What It Looked Like Before
==========================

By using the jQuery `extend()` method, you can create many isolated components (mixins) that do one job. Then, you can combine them into one object which implements all of the mixins. The JavaScript code would have done something like:

    var TextBox.prototype = $.extend({ ...some object with functions and state....}, CanDoExtraStuff, CanDoOtherCoolStuff);
    

How I Use Decorators
====================

I used a combination of tchniques from the TypeScript offical documentation

*   [Mixins](https://www.typescriptlang.org/docs/handbook/mixins.html "mixins")
*   [Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html "decorators")

TypeScript decorators allow you to change a class/prototype, function, etc. at run-time by adding annotations in your code. (Angular 2 uses decorators quite a bit - @Component, etc.) Decorators allow changing a class or function in a clean and understandable way. Here's the code I'm using to build a mixin decorator:

    // In Your Decorator File
    export default function (...mixins: any[]) {
       return function (base) {
          mixins.forEach(mixin => {
             Object.getOwnPropertyNames(mixin.prototype).forEach(name => {
               base.prototype[name] = mixin.prototype[name];
             });
          });
       }
    }
    

    // In Your Component / Module File
    @ApplyMixins(CanDoExtraStuff, CanDoOtherCoolStuff)
    export default class TextBox implements CanDoExtraStuff, CanDoOtherCoolStuff {
       ...code...
    }
    

Conclusion
==========

It's a simple change, but it makes applying mixins more succinct and breaks the dependency on jQuery to do the assignment. What do you think? Would you prefer using decorators to jQuery extend?