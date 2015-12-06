---
layout: post
title: What is all this Clean Architecture jibber-jabber about? - Part 2
---

In the last [post](http://pguardiola.com/blog/clean-architecture-part-1), we walked through the _Clean_ part, giving a summary of some concepts that you should know if you wish to write _well-crafted code_.
If you want to go more deeper I encourage you to review other principles and patterns that have been explained and discussed for so long, like [GRASP](https://en.wikipedia.org/wiki/GRASP_%28object-oriented_design%29) and the ones included in the [Gang Of Four](http://c2.com/cgi/wiki?GangOfFour) [Design Patterns Book](http://c2.com/cgi/wiki?DesignPatternsBook).
In this post, we are going to mention some layered architectures, explaining what they have in common and their advantages and disadvantages. Furthermore, we'll go deeper, describing Hexagonal Architecture and comment some of its potential downsides.
So, less talk and more action!

## Architectures: Different names, same philosophy

As you may know, over the last several years many different architecture approaches have appeared. These include: [Onion Architecture](http://jeffreypalermo.com/blog/the-onion-architecture-part-1/), [Hexagonal Architecture](http://alistair.cockburn.us/Hexagonal+architecture) (also known as Ports and Adapters) and the popular [Clean Architecture](https://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html), among other layered architectures.
Though these approximations have their own fancy names and could seem completely different, they share a lot in common. In fact, its main intent is the same:

> _Achieve a high-level separation of concerns by layering_.

Ultimately, as we've seen in the previous article, _Clean Architecture_ represents a group of _best practices_ that allow us to implement code that _rocks_, which means, spending time on what is important:

> Building awesome applications that people love.

### What are the main benefits of going that way?

All the above approximations create software that are:

#### Independent of frameworks

Frameworks are considered _services_. That is to say, we can focus on the business logic (what makes our app different from others) being agnostic to the outside world so that you have the possibility of choosing whatever framework you want and the most important thing:

> The frameworks that you use don't end up being your application itself.

Here, when referring to _frameworks_ we also include other low level details such as _GUIs_, _databases_, _network services_, etc. In short, whichever part of our application that we might have to change without modifying our inner core rules. That give us the chance of deferring the decision of which technology to choose until when we really need it.

#### Testable

Our code becomes extremely easy to test because our domain logic is decoupled from the implementations details of the outside world. That results in quicker tests since we are able to validate our system in isolation (without loading the database or making real HTTP requests) using [Test doubles](http://blog.8thlight.com/uncle-bob/2014/05/14/TheLittleMocker.html) for resolving our dependencies.
In addition to get immediate feedback that things work as expected, there are other motivators for creating tests like _Document the behaviour of the system_, _TDD_ or, the one that I like the most, _Enable_ [_Refactoring_](http://martinfowler.com/books/refactoring.html).

#### Easy to understand

Ultimately, we should strive to make our code read like a story. Back in the 90's, _seniority_ was acquired depending on how clever you were writing code. Some _senior_ developers wrote software that just a few could understand, proving how smart they were. Nowadays, we work in large teams with a bunch of programmers changing the same base code and generating thousands of lines every day. Imagine for a moment that you were in this situation, would you be able to understand everything written that way? What about your new teammates or apprentices? Do you think would be fair for them? Right now, that's disrespectful and unacceptable.
As you can imagine, _Architecture_ plays an important role in all this. We should structure our applications so that you can understand what they do at first glance. One way to achieve that is within _vertical slicing_.

___

Summarizing, the above benefits lead us to aim for architectures that contain the following attributes:

#### High Maintainability

_A maintainable application is one that increases technical debt at the slowest rate we can feasibly achieve._

In the beginning of a greenfield project you have everything under control. As time passes, bugs start appearing, adding new features implies changes in different parts of your code... So, ultimately, applications get tougher to work on.

A good architecture in advance can help prevent such potential problems.

#### Low Technical Debt

_Technical debt is the debt we pay for our "bad" decisions, and it's paid back in time and frustration._

The debt can be thought of as work that needs to be done before a particular job can be considered complete or proper. If the debt is not repaid, then it will be hard to implement changes later on and we will end up doing _hacks_ and work-arounds everywhere.

So we need to reduce as many "bad" decisions as possible and as soon as possible.

Basically, in order to produce maintainable applications we need to make them easy to change and, in order to do that, we must follow the well-known design principle:

> Identify the aspects of your application that vary and separate them from what stays the same.

### But, all that glitters is not gold...

Like everything in life, _all that glitters is not gold_. Below we are going to explain some drawbacks that we need to take into consideration when architecting our apps.

#### Higher level of complexity

Normally, more levels of abstraction mean more difficult to understand. Multiple layers imply more files to handle, which means that following the flow of our use cases becomes harder.

#### Over-engineering?

There are a lot of scenarios where _we aren't gonna need that level of adaptability_. Especially in mobile applications in which the domain logic is really close to the view.
Prematurely creating that kind of indirection and isolation is usually a waste of time and it may be a bit overkill.

## Hexagonal Architecture

### Beginnings

The name of _Hexagonal architecture_ comes from [Alistair Cockburn's article](http://alistair.cockburn.us/Hexagonal+architecture). As he explains in his post, its main purpose is:

> "Allow an application to equally be driven by users, programs, automated test or batch scripts, and to be developed and tested in isolation from its eventual run-time devices and databases".

### Ports and Adapters

As you may know, Hexagonal architecture is also called _Ports and Adapters architecture_. In fact, those terms are the essence of the architecture.

The number of ports (sides) is arbitrary and it depends on the application. The key point is that the business logic is at the center, inside the _hexagon_, and each side represents some sort of a conversation the application needs to have with the outside world.

#### Ports

_A port can be viewed as an entry point or a contract, provided by the core logic. It defines the functionality that the application offers_.

#### Adapters

_For each external device there is an adapter that converts the API definition to the signals needed by that device and vice versa._

In brief, ports translate to _interfaces_ and adapters to _implementations_.

Ports and adapters, when implemented, come in two flavors, namely _primary_ and _secondary_ or _driven_ and _driving_. The distinction between each lies in who triggers or is in charge of the conversation. That is, outside-in the hexagon and the other way around, respectively.

Summing up, outside the hexagon you have layers of ports and adapters that take requests from the outside world into the application. The resulting message from the application is then passed back through this layer of ports and adapters as an appropriate response.

### The dependency rule

_The dependency rule_ is key to making this kind of architectures work properly. So we need to understand it very well in order to apply it correctly.

This rule says:
_Source code dependencies can only flow from outside, in. Nothing in an inner layer can know anything at all about something in an outer layer._

When data is moving in, dependencies are not big deal. We need to be more careful when the dependencies are going out. But remember, we've already learnt how to deal with this complexity. We only need to apply _Inversion of control_ and problem solved!
Indeed, we can achieve it by using interfaces. These allow our layers to inform other layers how they will be interacted with, and how they need to interact with other layers. It's up to the other layers to implement these interfaces. In this way, we are letting our inner layers command how they are used and our dependencies keep going in one direction, inwards.

### Pros and cons

The same advantages and disadvantages described before apply in this specific approach.

Personally, I'd like to highlight the following:

#### Pros

* _Ports and adapters are interchangeable_.
That allows us to swap an adapter with a different implementation that conforms to the same interface. Although this might seem rare, you might need to replace them for different reasons, like testing or because one library gets deprecated, for example.

* _Defer decisions until the very end_.
You are able to code the logic of your application without worrying what aspect will have the user interface or which database you will choose. That's really powerful because different people can be working on different parts without affecting each other and, if they follow the contracts, when gluing everything together, it will work. 

* _Implement features faster_.
When you know how to convert requests and responses as they come and go from the outside world it is really easy to implement new features. The architecture facilitates you the separation of concerns, which means you can concentrate on one specific task at a time and you develop faster.

#### Cons

* [_YAGNI_](https://en.wikipedia.org/wiki/You_aren't_gonna_need_it).
The main concern of _Hexagonal architecture_ is that you could end up with interfaces with only one implementation and you need to be careful because the architecture holds you to the usage of _ports_ and _adapters_ in every layer. But, as always, you need to be pragmatic and don't forget to [KISS](https://en.wikipedia.org/wiki/KISS_principle)!

## Conclusion

As we've seen, pros beat cons! So, it's clear that having an architecture for our apps is something that we must always take into consideration. Call it what you want but don't forget that it's the only way to achieve a maintainable, testable application with low technical debt.

___

In the next and final article we'll get our hands dirty. I'll present **Catan Architecture**, a hexagonal architecture for Android. We'll analyze the overall structure, how the data crosses the boundaries of the different layers and all the rest of the implementations details by reviewing a sample project.
Who wants to play [Catan](https://en.wikipedia.org/wiki/Catan)? Don't settle for less!

P.S I hope you're enjoying the series but I would love to hear your thoughts! So feel free to leave a comment ;)