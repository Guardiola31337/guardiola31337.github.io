---
layout: post
title: What is all this Clean Architecture jibber-jabber about? - Part 1
summary: We will review concepts like SOLID, coupling and cohesion, abstraction, vertical slicing and some design patterns and principles all widely seen in each different architecture approach.
---

Lately, there are a lot of people in the Android community, talking about _Architecture_, especially about _Clean_. That's good news!
I'm pretty sure some of you are familiar with terms like _layers_, _Ports and Adapters_, _boundaries_, etc. but others aren't.

Although _Architecture_ is not new in _Mobile development_, (E.g. [Forgetting Android](http://www.slideshare.net/flipper83/forgetting-android?qid=d63a33ce-30c4-43a4-a049-41ce91b71980&v=default&b=&from_search=2) by [Jorge J. Barroso](https://twitter.com/flipper83) and [Software Design patterns on Android](http://www.slideshare.net/PedroVicenteGmezSnch/software-design-patterns-on-android) by [Pedro Vicente Gómez](https://twitter.com/pedro_g_s)), we developers, after years of hard work, are realising that it's an important aspect if we want our apps to succeed.

So... What's the problem??

I'm still seeing _poker-faces_ and hearing comments like _"It's too messy!"_ when discussing the topic with coworkers and people from the community.

Because of that, I think, it's a good moment to explain clearly the basics of every _architecture_, its parts in common and why it's so important in every piece of software.
Finally, I'll put my two cents in, describing everything that I've found along the way implementing another architecture's solution, which I've been working on recently:
**Catan Architecture**, a hexagonal architecture for Android.

In this series of posts, we will review concepts like [_SOLID_](https://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29), _coupling_ and _cohesion_, _abstraction_, _vertical slicing_ and some _design patterns_  and _principles_ all widely seen in each different architecture approach.
We will learn how to build a true [_Gold Field_](http://catan.wikia.com/wiki/Gold_Field) in Android. We will define the structure of the [_hex_](http://catan.wikia.com/wiki/Resource_hex), explaining the concepts that support this layered architecture. We will carefully show the way to achieve a maintainable, testable application with low technical debt, analyzing the pros and cons of these kinds of architectures by reviewing **Catan Architecture**'s sample project.

## Motivation

In recent months, lots of _Android architectures_ talks, posts and implementations have appeared; [Flux Architecture on Android](http://lgvalle.xyz/2015/08/04/flux-architecture/) by [Luis G. Valle](https://twitter.com/lgvalle), [Effective Android Architecture](https://speakerdeck.com/richk/effective-android-architecture-mwdc) by [Richa](https://twitter.com/richa123), just to mention a couple.

[Architecting Android](http://fernandocejas.com/2015/07/18/architecting-android-the-evolution/) is cool and trendy!

In order to clarify why we should care about architecture and not blindly apply one of them, I decided to write this series of articles and share my thoughts.

Besides, I'm a strong believer in collaborative learning so I'd like to remind everyone to join the discussion below and share your views. It's the best way to learn from others and to improve our craft!

## The basics

I always like to start by explaining the core main concepts of the topic which I'm going to talk about. I think it's good to review the ideas that support the subject matter and define them in a simple, easy to understand and remember, way.
This time is not going to be the exception, so let's go!

### SOLID

In my opinion, this mnemonic acronym is the key to master object-oriented programming and design. These five Must-Know principles form the basis of which we call **_Clean Code_** nowadays.

Don't forget the following quote by [Uncle Bob Martin](https://twitter.com/unclebobmartin) extracted from his [book](http://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882):

> "The ratio of time spent reading (code) versus writing is well over 10 to 1 ... (therefore) making it easy to read makes it easier to write."

That is really well explained with another quote from a [friend of mine](https://twitter.com/pedro_g_s):

> "Write code for your mates not for the machine."

I think, all is about striving to write code with semantics applying the utmost rigour and being a pragmatic and disciplined programmer.
So, we can't go forward without reviewing and understanding clearly the pillars on which _Clean_ is built.

#### Single Responsibility

_A class should have only a single responsibility_

The single responsibility principle states that every class should have responsibility over a single part of the functionality provided by the software.
Robert C. Martin defines a responsibility as a reason to change, and concludes that a class or module should have one, and only one, reason to change.

#### Open/Closed

_Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification_

That is, such an entity can allow its behaviour to be extended without modifying its source code.

#### Liskov Substitution

_Objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program_

The Liskov substitution principle states that, in a computer program, if S is a subtype of T, then objects of type T may be replaced with objects of type S (i.e., objects of type S may substitute objects of type T) without altering any of the desirable properties of that program.

#### Interface Segregation

_Many client-specific interfaces are better than one general-purpose interface_

The interface segregation principle states that no client should be forced to depend on methods it does not use. ISP splits interfaces which are very large into smaller and more specific ones so that clients will only have to know about the methods that are of interest to them.

#### Dependency Inversion

_One should “Depend upon Abstractions. Do not depend upon concretions_

The dependency inversion principle concerns itself with decoupling dependencies between high-level and low-level layers through shared abstractions.

DI states:
1. High-level modules should not depend on low-level modules. Both should depend on abstractions.
2. Abstractions should not depend on details. Details should depend on abstractions.

### Inversion of Control, Dependency Injection and Dependency Inversion

As a developer, I know, you probably mix up the concepts _Dependency inversion_, _Inversion of control_ and _Dependency injection_ easily. All of them sound similar but they don't mean the same and you should learn their differences well.

Following I describe the rules of thumb that helped me to master those concepts.

#### Inversion of Control

The simplest way to remember what Inversion of control refers to and never forget it is to stick it with the [Hollywood principle](https://en.wikipedia.org/wiki/Hollywood_principle).
Now, when I talk about Inversion of control the first thing that comes to my mind is:

> "Don't call us, we'll call you."

#### Dependency Injection

_Dependency injection is a software design pattern that implements inversion of control for resolving dependencies_

The rule that I apply with Dependency injection is to think in the _container_ and how it works. That is, giving an object its instance variables when needed "_auto-magically_".

**UPDATE**

Just to clarify "_auto-magically_". Here, I'm referring to how a dependency injector (container) works. It's the technique that I use in order not to mix it up with IoC and DIP.
As my colleague [Tomás Ruiz-López](https://twitter.com/tomasruizlopez) points out in the comments, dependencies could be provided through the constructor or through setter methods (among others) and in order to provide those dependencies, we can use a DI framework, or other solutions like a Service Locator or even a Factory pattern.

#### Dependency Inversion

Finally, regarding to _Dependency inversion_, the following definition is the one that I use and, in my opinion, the one that best represents its general meaning:

> "Dependency inversion is about the level of the abstraction in the messages sent from your code to the thing it is calling."

To sum up:

> "DI is about wiring, IoC is about direction, and DIP is about shape."

All the details are carefully described in this enlightening [post](http://martinfowler.com/articles/dipInTheWild.html).

### Abstraction

One of the most fundamental concepts of OOP is _abstraction_. As you may have noticed, it appears in almost every principle in programming. So, it's really important to understand its meaning well.

_Abstraction is the concept of describing something in simpler terms, i.e. abstracting away the details, in order to focus on what is important_

It's used to decompose complex systems into smaller components.
From my point of view, abstractions are a really good way to describe the metaphors that we are trying to tell in our programs.

As you know, abstraction in Java is achieved by  using `interface` and `abstract class`. An _interface_ or _abstract class_ is something which is not concrete.

You need to be careful because in other programming languages abstraction is reached using other techniques or keywords but, remember, the overall definition is still the same.

#### _Program to an interface, not an implementation_

> "Program to an interface", really means "Program to a supertype".
– [Head First Design Patterns](http://www.amazon.com/gp/product/0596007124?ie=UTF8&tag=fatagnus-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=0596007124)

What this abstraction-related best practice means is to focus your design on what the code is doing, not how it does it.
There are several benefits of this approach like maintainability, extensibility and testability.

Here, just to clarify it, when talking about _interface_ we are referring to the general concept not to the Java reserved word.
With this in mind, it seems _abstraction_ and _interface_ are the same, but that's not always true. Check this [post](http://blog.ploeh.dk/2010/12/02/Interfacesarenotabstractions/) if you want to go deeper into the subject. 

### Coupling and Cohesion

Both concepts occur together very frequently and we definitely should know them well.

#### Coupling

_Coupling is the manner and degree of interdependence between software modules; a measure of how closely connected two routines or modules are; the strength of the relationships between modules_

#### Cohesion

_Cohesion refers to the degree to which the elements of a module belong together. Thus, cohesion measures the strength of relationship between pieces of functionality within a given module_

We need to strive for _low coupling_ and _high cohesion_ in software development. Because, low coupling (decoupled) is often a sign of a well-structured computer system and a good design, and when combined with high cohesion, supports the general goals of high readability and maintainability.

### Vertical slicing

The concept of _Vertical slice_ comes from the _Agile_ movement. It's mostly used in _Scrum_ terminology where the work is planned in terms of features (or stories).

_Horizontal slicing_ is the traditional approach in which you use architectural layers (such as DB, network, security, middleware application, UI...) as a method of decomposition of features.
_Vertical slicing_, on the contrary, breaks it down into small stepwise progressions where each step cuts through every slice.

Basically, it means to develop from the concrete _use case_ you are focused on within an end-to-end perspective in mind. That allows you to implement one _user story_ at a time generating value progressively and creating applications incrementally, obtaining quick feedback for free, feedback that'll be really useful to improve your product.

That is closely related with how you architecture an app and we'll go deeper in the next posts of this series.

## Conclusion

It may sound boring, but the above concepts are the main beams of producing high quality code in OOP environments.
Ultimately, like everything in life, in software architecture there is no _silver bullet_ either.
For me, it's a work in progress in which you are learning by doing, getting feedback from others and, consequently, improving your knowledge through discussions of different ways of doing the same thing.
_Clean Architecture_ is all about common sense, pragmatism and professionalism. Being aware of the trade-offs made and always being smart in order to deal with them properly.

___

In the [next](http://pguardiola.com/blog/clean-architecture-part-2/) segment, we'll dive into the _architecture_ part in detail. We'll analyze the big picture of some layered architectures, explaining their pros and cons and finally we'll describe _Hexagonal architecture_ in detail.
If you expect to see _Catan Architecture_'s code you'll need to wait for the third and final article. Be patient and stay tuned!

P.S. Quick reminder! I encourage you to leave a comment below explaining your thoughts. I'm looking forward to hearing from you. Let's all debate!