---
layout: post
title: Solid Hexagonal Functions
cover: cover.png
date:   2018-12-11 13:00:00
categories: posts
---

# The Truth 

The truth is often very restrictively visible to us. It is like we are always looking through the keyhole, and always wondering what is behind the door. So we made theories about it, we argued about the theories, and we fought each other to prove we were right. 

Most theories had a small portion of truth from a certain point of view, but largely consisted of imagination and conceptual structures.

Same thing happened in the programming world. We have been searching a way to design our software better and we did now know how. We started by programming machinery to perform mechanical routines, then we had the first computer and the machine code. Then we had the assembly code, then procedural programming language, then object-oriented and functional programming.

Funny enough, functional programming is very ancient, older than the first computer. The concept of functions came from mathematics. The truth of the objective world has not been evolved, we have. We learned and understood more everyday.

There have been a lot of principles and architectures how we could design our software better. All of them had their own merits, because they genuinely tried to solve a problem, from their perspectives, their views from the keyhole. It should not be a surprise that many of them could be combined and complemented each other and converged to even better ways to solve a problem.

# The Trinity
![Trinity](/images/trinity.png)

Here is my attempt to combine SOLID Principles, Hexagonal Architectures and Functional Programming. The first part is a brief introduction of all these concepts, and the second part is fun part where these concepts come together. 

# SOLID Principles
SOLID Principles historically applied to object-oriented programming only, as the best practices how classes should be organised.

## Single Responsibility Principle (SRP)
*"A class should have one, and only one, reason to change.", [Robert C. Martin](http://blog.cleancoder.com/), 2006, Principles of Object Oriented Design.*

Take a look at this class:
```java
class TransactionAppender {
    void appendTransaction(String transactionInXml) {
    }
}
```

It looks like it has one responsibility, which is to append a transaction that is in XML format. What are the reasons to modify this class? There are two actually:

* The XML format has changed
* The transaction structure has changed, e.g. a new field

So where does the reason of change come from? I believe the reason to change a class is to address a concern. So in my opinion, if a class addresses one and only one concern, it should naturally fulfil the Single Responsibility Principle and separation of concerns.

The class in the previous example can be split into two to conform to the Single Responsibility Principle:
```java
class TransactionAppender {
    void append(Transaction transaction) {
    }
}
class TransactionTranslator {
    Transaction translate(String transactionInXml) {
    }
}
```

## Open Closed Principle (OCP)
*"Software entities … should be open for extension, but closed for modification.", Bertrand Meyer, 1988, Object Oriented Software Construction.*

There are two statements in this principle:

* A module will be said to be open if it is still available for extension. Taking the previous example, the behaviour of `TransactionAppender` can be extended to support `Transaction` in XML format by a class that use `TransactionTranslator` to translate the XML String to `Transaction` and then append the `Transaction` using `TransactionAppender`.
* A module will be said to be closed if is available for use by other modules. That implies class `TransactionAppender` can be re-used by taking the compiled class and does not require any modification of the re-used class.

## Liskov Substitution Principle (LSP)
*"Let Φ(x) be a property provable about objects x of type T. Then Φ(y) should be true for objects y of type S where S is a subtype of T.", Barbara Liskov, Jeannette Wing, 1994, A behavioral notion of subtyping.*

Ok, calm down. It could be translated as this:
*"objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program."*

Let's say you have an interface for sorting numbers:
```java
interface Sorter {
    int[] sort(int[] numbers);
}
class BubbleSorter implements Sorter {
    int[] sort(int[] numbers) {}
}
class MergeSorter implements Sorter {
    int[] sort(int[] numbers) {}
}
```

If `Sorter` is used in a program, it can be `BubbleSorter` or `MergeSorter` and they will return the same result. One can replace the another while keeping the correctness of the program. The difference is in the implementation details while the design contracts are the same.

## Interface Segregation Principle (ISP)
*"Many client-specific interfaces are better than one general-purpose interface.", [Robert C. Martin](http://blog.cleancoder.com/), 1996, The Interface Segregation Principle.*

Have a look at these classes. 

```java
class Person {
    String name;
    String employeeId;
    String customerId;
}
class CustomerRegistry {
    void register(Person person)
}
```

The `Person` class is a mixture of a person, an employee and a customer. Unless the business model really treats three of them as the same entity, this class already has violated the principle. And the class `CustomerRegistry` demonstrates how this general-purpose class is used for an operation specific to a customer. The client using this class is forced to be aware of `employeeId` while customer should be the real entity. 

## Dependency Inversion Principle (DIP)
*"One should "depend upon abstractions, not concretions.", [Robert C. Martin](http://blog.cleancoder.com/), 1994, Object Oriented Design Quality Metrics: an analysis of dependencies.*

Abstractions are the specification of the function and concretions are the implementation. When an API is designed by contracts, i.e. specifying what it does, it is enough for the users to use the API without the need to understand how the API was implemented.

Using the example of sorters previously mentioned, the `Sorter` is the *what* while `BubbleSorter` and `MergeSorter` are the *hows*. And certainly the users of sorter API do not need to know if it is bubble or merge sort being used. It is irrelevant in terms correctness and system semantics.

# Hexagonal Architecture
![Hexagon](/images/hexagonal.png)

[Hexagonal architecture](https://blog.ndepend.com/hexagonal-architecture/) modelled application in layers that separate concerns and make testing easier. Hexagonal architecture is also called Ports & Adapters. A very similar pattern is called the Clean Architecture.

## Domain Layer
The core of the architecture is the application domain. Strictly speaking, the domain is the pure logic that models the domain entities and functions. It is the cognitive behaviour, the decision maker, and the brain of the application. It should not be tied to any technology choice.

This is where the [Behaviour-driven development (BDD)](https://medium.com/@TechMagic/get-started-with-behavior-driven-development-ecdca40e827b) shines. The developers can ensure the application make the right decision by driving the code from behaviour test cases, without the clutter of mocking REST, JSON, XML, or any particular technology. 

## Integration Layer
The layer around the core (Domain) is the Integration Layer. This layer consists of all the technology choices but none of the application logic. The operations are typically data transportation, transformation, splitting and merging. If the domain is the brain, then integration is the body. Integration layer does not make any decision that concerns the business.

## Ports
Ports are the interfaces between the domain and the integration. Ports define how an external data get into the domain ("Primary Ports"), and how an internal data get out of the domain ("Secondary Ports").

Primary Ports are the perceptive and sensory parts of the body (eyes, ears, the mouth, the nose, etc.), and Secondary Ports are the motor parts of the body (torso, arms, legs, etc.).

Ports are defined by the domain. It is the domain who decides how to interact with the external layer. The integration layer has to adapt to the domain, and not the other way round. And therefore the implementation of Ports is named the "Adapter".

## Adapter
Adapters are the implementations that fulfil the contracts defined by the Port interfaces. 

If the domain intends to notify that an order is placed, the Email Notifier Adapter would send an email for the order confirmation; the Inventory Integration Adapter would contact the Inventory system to schedule the delivery. But all adapters here are notification of a placed order in their own ways.

If Ports are the "What", then Adapters are the "How".

# Functional Programming (FP)
Functional programming organised an application as immutable data structures and stateless functions. The simplest expression is that if `y = func(x)` then `func(x)` always returns `y`.

It is predicated on the states being immutable and functions being stateless. With the preconditions fulfilled, the states and functions can be shared and reused with no side effect, no concerns on concurrency, and no consistency issue.

## Immutability
To ensure the function always return the same value given the parameters, the data structures used for input and output must be immutable. Otherwise the function is no longer *always* if the input and output can be tampered. 

Here is a simple example of what functional programming is not:
 
```java
java.util.Date time = convert(timeAsString);
time.setTime(System.currentMillis());
```
The class `java.util.Date` is mutable. As a result, even the variable `time` was a representation of `timeAsString`, the next line updated the intrinsic of `time` that it no longer was the representation of `timeAsString`.

There is no way to guarantee function `func(x)` always returns `y` if x or y can be modified. Moreover, x and y cannot directly or indirectly reference to a mutable structure.   

In the Java collection library, there are a lot of mutable classes. A mutable collection of immutable objects is still mutable. This is where the persistent collection come to rescue, as every *modification* to it returns a new collection.

There are a few tools to help out creating immutable classes and persistent collections in Java.

Immutable classes
* [Immutables.io](https://immutables.github.io/) - 
* [AutoValue](https://github.com/google/auto/blob/master/value/userguide/index.md)
* [Lombok](https://projectlombok.org/)

Persistent collections
* [PCollections](https://pcollections.org/)
* [Vavr](https://github.com/vavr-io/vavr)

## Stateless Functions
To guarantee function `func(x)` always returns `y` given x and y are immutable, there are still a few restrictions

* There is machine time-based decision. 
* There is no randomised behaviour. 

Any attempt get a time on the box, or get a randomised value, would result in the function returning different values at different times, given the same parameter. Then the function is not deterministic, and therefore is not a stateless function.

It is common that external scheduler or randomiser is used to pre-generate the values as parameters so the function can stay stateless.

# Will it blend? That's the question
Now we have got SOLID Principles, Hexagonal Architecture and Functional Programming here. You may already realised that some of them are effectively the same concepts but described in different ways. There are some interesting chemistry happening when we blend them together. 

Are you ready?

## Port as single-responsibility functional interface
A Port interface with many methods is not really helping anyone. If you were to just swap the behaviour of one of the methods, then you will end up inheriting the original implementation and override the one that you want to change.

On the other hand, if each Port interface has only one method, then it eliminates the need of inheriting an implementation and overriding just the one you want to change. Your code change is smaller and so is the risk. Your test case is smaller and you work less for the same business value. It also offer the maximum flexibility of behaviour combinations. It is indefinitely scalable as you could define as many Ports as the application requires, without creating a God class ever.

Also it is much easier to fulfil Single Responsibility Principle if there is only one single method - as long as your method performs only one thing. There is only one reason to change this interface, which is to change this one behaviour.

Moreover, one method in an interface in Java is the functional interface. It can be provided as lambda expressions, and therefore no need to use anonymous inner class. Also as it is a single-method interface, the Port can be stacked with other Port to form more complex behaviours. This is the maximised way to re-use Ports as stateless functions.

## Open-closed Immutable states
Immutable states are safe to be re-used or inherited, in other words, open for extension. It is resistant to abuse and is consistent within itself.

You certainly do not need re-compile to use an immutable state, so it is by nature closed for modification.

## Open-closed Stateless Functions
Stateless functions have no side effect and result is always the same given the same input at all time. They are effectively immutable behaviours. And because of that, they are predictable and reliable when stacked together as a complex behaviour. They are open for extension. Modifying one of the functions does not require any re-compilation, which in other words, closed for modification.  

## Substitutable Adapter Functions
As Ports are defined by the domain, their Adapters are apparently substitutable with altering any desirable behaviour of the domain, simply because all Adapters of the same Port are fulfilling the contract defined. It is the same as the fact that you could swap the implementation of an interface by another.

## Segregated Port for each client as Function 
As Port is now the single-responsibility functional interface, it is also true that if a client would need to use a Primary Port differently, a Port functional interface will be tailor-made for the client. It also implies overloading methods will not appear in Port interface, as different method signature of the same behaviour will be defined in its own Port interface.

## Injecting dependency on Port as Function
The Integration layer should only depend on Primary Port interface and not the implementation. Same principle applies to Domain layer depending on Secondary Port interface only.

By doing that, we have separated the concerns of application domain logic and the technology choice for the Adapters. So if we were to change the domain logic, the code is only changed in the domain; if we were to change the technology, e.g. from REST to JMS, then only the integration layer requires modification. Again, the change is smaller and therefore the risk is lower.

# Where does Object-oriented Programming sit?
Shifting the paradigm to Functional Programming does not mean the end of Object-oriented Programming. In fact, Object-oriented programming improves code quality of Functional Programming. Even a small method contains a portion of Procedural Programming. Assembly and machine do still exist but they sit at lower levels. They all exist and we should use them where we get the most benefits.

Object-oriented programming still has its value even if Functional Programming is used as the major paradigm. Here is a brief explanation.

## Inheritance of Immutable states
Data structures are great showcases of encapsulation. If you were to implement a Hashmap, you still need to protect data integrity protection that no external code can hack and break the correctness. The other reason is noise reduction that you only need to expose what is needed for the client, and therefore making your data structure neat and clean. Also modifying code internal does not affect the client, and thus lower the risk.

Inheritance of immutable states is certainly a valid usage, given the subclass can be use as the equivalence of its parent class (Liskov Substitution). However, most programmers would choose [Composition over Inheritance](https://proandroiddev.com/composition-over-inheritance-in-kotlin-way-fe341159bf1c) these days if possible. 

## Inheritance of Functions
This is the perfect marriage of functional and object-oriented programming. Functional interface defines *what* it does, and the implementation details *how* it works to fulfil the design contract.

As long as the implementation still fulfil the design contract (the What), it does not matter if one implementation inherits another. We already have covered depending on Port as functional interface. And by applying these practices together, the programmers have the freedom to use inheritance of function without consequence.

One of the usual reasons function implementation may inherit another is different performance characteristics, as the externally perceived behaviour (design contract) has to be the same. 

# Summary
I would like to reference to this amazing picture of programming evolution from the blog of ["So You Want to be a Functional Programmer"](https://medium.com/@cscalfani/so-you-want-to-be-a-functional-programmer-part-5-c70adc9cf56a) by Charles Scalfani..

![Evolution](/images/evolution.png)

The programming paradigm has evolved but we still have proportions of each previous paradigm in our code and design.

Combined with the SOLID Principles and the Hexagonal Architecture, Functional Programming is stronger than ever. And I hope instead of debating whether Object-oriented or Functional Programming is better, we should make use of both to get the best values from them when it is appropriate.

To feel the true power of the Trinity (SOLID, Hexagonal, Functional), I would recommend anyone to try to solve a problem (e.g. in [HackRank](https://www.hackerrank.com/) or [Codility](https://app.codility.com/free-trial/?utm_campaign=google-brand&utm_medium=cpc&utm_source=google&utm_content=free-trial)) using these three practices together. Then compare the ease of unit testing, number of lines in a file, separation of concern, ease of extension, readiness for evolution, to your original habitual approach.

Let the Trinity guides you, and may the truth be with you. 
