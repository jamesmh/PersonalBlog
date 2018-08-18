---
title: 'TypeScript Decorators: Bind Constructor Parameters Automagically'
tags:
  - constructor
  - decorators
  - typescript
url: 189.html
id: 189
categories:
  - TypeScript
date: 2017-03-06 19:24:43
---

The more I use TypeScript the more I realize how powerful it really is. TypeScript decorators are one of those fantastic features that let you do things like automate common tasks and create isolated modules that can be reused. TypeScript constructor decorators allow you to do some really fancy stuff. Keep reading.

Simple Example
==============

One of the use-cases I built for a project is a class decorator that automates binding a constructor's parameters as properties on the new object. Here is code that is pretty typical - without using decorators.

    class Person {
       private name: string;
       private email: string;
    
       constructor(name: string, email: string){
          this.name = name;
          this.email = email;
       }
    }
    

We want to enable make our class to assign the constructor parameters as properties of the instance.

    @BindConstructorParametersAsProperties //Or whatever you want to call it ;)
    class Person {
       private name: string;
       private email: string;
    
       constructor(name: string, email: string){}
    
       public printEmail(){
          console.log("My email is: " + this.email);
       }
    }
    

Notice how the constructor is empty? The decorator will automatically bind the constructor's parameters as part of the class instance. In the example above, `printEmail()` will work as intended. This utility doesn't do that much. But it's better to have many small re-usable utilities than huge monolithic ones - and this is hopefully one you find helpful.

Code
----

[GitHub Gist for the decorator code](https://gist.github.com/jamesmh/1554619a41d3bbfafa06e55016a0bd0d "gist")