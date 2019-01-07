---
layout: post
title: The Myth of Read-write Multi-threading
cover: cover.png
date:   2018-11-07 00:00:00
categories: posts
---

## Background

One of the most common patterns in an application is that you have your [Aggregate](http://thepaulrayner.com/blog/aggregates-and-entities-in-domain-driven-design/) in the domain, and your Aggregate is being read and is being requested to be updated, at the same time.

There are at least two flavours of updates already:

* Incrementally updated. The content of the next version of Aggregate depends on the current. It also implies the decision of update content requires the read operation. 
* Independently updated. Each update of the Aggregate has no relation nor dependency. It does not need a read operation. 

## Independent updates

Since independent updates does not require a read operation, so we only need to replace the reference atomically. 

### Volatile keyword

The simplest implementation is the use of Java `volatile` keyword:

```java
public class Item {
    private volatile String description;
    
    public void setDescription(String newDescription) {
        this.description = newDescription;
    }
}
```

The [volatile](https://www.baeldung.com/java-volatile) keyword is treated like a flush operation that all non-volatile fields will be up-to-date in main memory before the `volatile` write happens. Given this semantics enforced by the Java Memory Model, the update of a `volatile` field is atomic and thread-safe.   

### Atomic classes

Another approach is the use of Atomic classes under `java.util.concurrent.atomic`. However, it is recommended to use `final` keyword on the Atomic class field reference to ensure no one could replace the Atomic class instance. 

```java
public class Item {
    private final AtomicReference<String> description = new AtomicReference<>("initial");
    
    public void setDescription(String newDescription) {
        this.description.set(newDescription);
    }
}
```

## Incremental updates

Things get complicated if you need to read the current version of your Aggregate in order to generate a newer version. If thread A and B both get Aggregate version 10 and propose a version 11 based on 10, then one will overwrite another unknowingly.

You might notice I brought up the versioning concept here. Incremental updates imply a linear sequence of updates and version is effectively the position of the update in the linear sequence. Some domains may have the concept of branching, e.g. git, but it is out of scope in this discussion.

This would implies the application has to ensure that the request to update an Aggregate to version X is based on the version X-1.

### An attempt using synchronized block 

So the first thing that may come to your mind is, of course, `synchronized` block. `synchronized` block enforces the code inside can only be executed by the one thread that has obtained the lock. Does the following example work?

```java
public void addNumberToSum(int number) {
    int newSum = this.sum + number;
    synchronized (this) {
        this.sum = newSum;
    } 
}
```

No it does not, because the read operation is not synchronized and there is no guarantee your update to version X is based on version X-1.

### An improved attempt 

So the following should work, shouldn't it? 

```java
public void addNumberToSum(int number) {
    synchronized (this) {
        this.sum += number;
    } 
}
```

And yes it does! By the way, there are alternative implementations that are functionally equivalent. 

### Semaphore
 
This is the version using `java.util.concurrent.Semaphore` of one permit:

```java
public void addNumberToSum(int number) {
    semaphore.acquire(); // semaphone has only one permit
    this.sum += number;
    semaphore.release();
}
```

### Atomic classes

And for the sake of this `addNumberToSum` method, you can simply use `java.util.concurrent.atomic.AtomicInteger`:

```java
public void addNumberToSum(int number) {
    sum.addAndGet(number); // sum as AtomicInteger
}
```
### Performance

However, there are operations that only require reading but not writing. Are we happy to make all read and all write operations to wait for a single lock? Probably not.

How can we optimise the performance of read and write given we have correctness now? To design the solution, we need to understand the nature of the problem first.  

### Read-write ratio is the primary drive

It is important to understand if the application anticipates reading is more often than writing, or vice versa. In addition, the read-write ratio needs to be assumed, estimated or projected before an optimisation can be considered.

#### Reading is more frequent

Generally, it is more common to find reading more frequent than writing in an application. In this case optimisation is towards the read operation.

In theory, there is no need to acquire any lock or to enter `synchronized` block for a read operation when there is no write operation at the same time. It can be done by using the `java.util.concurrent.ReentrantReadWriteLock` :

```java
public void addNumberToSum(int number) {
    try {
        lock.writeLock().lock();
        this.sum += number;
    } finally {
        lock.writeLock().unlock();
    }
}

public int getSum() {
    try {
       lock.readLock().lock();
       return this.sum;
    } finally {
       lock.readLock().unlock(); 
    }
}

```

`java.util.concurrent.ReentrantReadWriteLock` keeps track of the read and write lock to ensure that write lock can only be acquired when all read locks are released and the write lock is not being held by another. It is worth to note that the finally block is necessary so the lock will be released even if exceptions were thrown. 

However, it still holds true that in order to generate a correct incremental update, you need to ensure you have the X-1 version. You may want a fail fast approach to save unnecessary locking, e.g. disabled flag. A approach similar to double-checked locking can be done like the following:

```java
public void sendEmail(String address, String subject, String content) {
    if (isEmailEnabledForUser()) {
       try {
           lock.writeLock().lock();
           if (isEmailEnabledForUser()) {
              // send email
           }
       } finally {
           lock.writeLock().unlock();
       }     
    }
}
```
If email is disabled, the operation ends immediately. Otherwise, write lock is acquired *first* then re-evaluate if the email is *really* enabled before the rest of the operation.

#### Writing is more frequent

Due to the nature of the linear sequence of incremental updates, there is not much you could do to optimise the write operation. However, if the read would take a longer time to finish (e.g. report generation), then it is preferred not to make a long queues for the writing. Also since read operation is infrequent, we could consider making an immutable copy of the Aggregate for the lengthy infrequent read operation, given if the copy operation is not more time-consuming than the read operation of the Aggregate. For example: 

```java
public HistoryImmutable getHistoryImmutable() {
    try {
       lock.readLock().lock();
       return new HistoryImmutable(this.history);
    } finally {
       lock.readLock().unlock(); 
    }
}

public void generateReport() {
    HistoryImmutable history = getHistoryImmutable()
    // generate report from copy
}

```

### An alternative approach I prefer

Are we still missing something? It seems going well so far! 

There is one option that was not considered. Is the immutable object is viable option?

Certainly immutable objects can be shared in multiple threads without even needing a lock. If each instance has a version then the incremental updates can be enforced too. 

At the end of the day, the immutable objects need a repository so all threads can get a up-to-date version at the time of request. That funny enough, implies the repository to look up the Aggregate would face the read-write multi-threaded problem we just mentioned. 

What if we don't? Then would mean the Aggregate will branch out for each thread, which is an entirely different problem.

#### Immutability

The main principle is that everything referenced by an immutable object should be immutable too, vertically deep to theoretically infinite. For example, an immutable object should never have a reference to a `java.util.Date` since it is mutable.

There are a number of ways to implement immutable objects:

* [Immutables.io](https://immutables.github.io/)
* [AutoValue](https://github.com/google/auto/blob/master/value/userguide/index.md)
* [Lombok](https://projectlombok.org/)
* Do it yourself

There are many discussions and comparisons on the Internet which I am not going to repeat. They are similar and I think the choice is down to personal preference.

Here is an example of an immutable object class using Lombok:

```java
import lombok.Getter;
import lombok.NonNull;
import lombok.RequiredArgsConstructor;

@RequiredArgsConstructor
public class User {
    @NonNull
    @Getter
    private final long version;

    @NonNull
    @Getter
    private final String username;

    @NonNull
    @Getter
    private final boolean enabled;
    
    public User setEnabled(boolean newEnabled) {
        if (this.enabled == newEnabled) {
            return this;
        }
        return new User(version + 1, username, newEnabled);
    }
}
```

Each attempt to modify the state will return a new object with a newer version, subject to the limit of Long max value.

#### Persistent collections

I also want to discuss collections as a part of the immutable objects. The Java collection classes under `java.util` are mutable. Even though there are helpers and subclasses that made it effectively immutable by throwing exceptions for all update operations, it still feels shoehorned to me.

At the end I found Persistent Collections. The persistent here means the previous version is preserved when modified. For example, if you add an item to a list, a new list will be returned. In other words, we have a clean immutable collections to use!

```java
   List newList = oldList.add(item);
   //oldList has n items and newList has (n + 1) items
```
I would recommend the following libraries that provides persistent collections. I personally strongly favour Vavr for the functional programming support, but I think PCollections will do fine if you only want persistent collections.

* [PCollections](https://pcollections.org/)
* [Vavr](https://github.com/vavr-io/vavr)

#### Version control

It is important to enforce version control to protect the integrity of the Aggregate. The write operation should check if the proposed change is of version "current + 1". When multiple threads attempt to update the same version of the Aggregate, the first update will be granted, while the rest will fail. The failed attempt requires reading the latest Aggregate version and re-apply the proposed change. It works exactly like a rebase operation in Version Control Systems.

Re-using the `User` class in this example, if the Aggregate is stored in a `ConcurrentHashMap` with `username` as key, it could work this way:

```java
    public void disablePlayer1() {
        ConcurrentHashMap<String, User> repo = new ConcurrentHashMap<String, User>();
        update(repo, new User(Long.MIN_VALUE, "player1", false));

        User expected, actual;
        do {
            User player1 = repo.get("player1");
            expected = player1.setEnabled(true);
            actual = update(repo, expected);
        } while (actual != expected);
    }

    private User update(ConcurrentHashMap<String, User> repo, User user) {
        return repo.merge(player1Enabled.user(), user, (current, proposed) -> {
            if (proposed.getVersion() == current.getVersion() + 1) {
                return proposed;
            }
            // Version mismatch, ignore update
            return current;
        });
    }

```

The update operation `disablePlayer1` will keep trying to update until it succeeds, and when it fails, it reloads the latest version and apply the change on top of it. On the other hand, the `update` method enforces that the next update must be of version "current + 1". However, a variation of this check could simply be (new version > current version), but any version jump bigger than one may be exposed to corruption.

#### Are we generating too much garbage?

The major criticism of using immutable objects in multi-threaded read-write operations is the vast amount of garbage generated.

It is true that this approach will generate more objects, but they are generally short-lived as they are replaced quickly by newer versions. As a result, they are mostly collected during the minor GC in the Eden space. The probability of these objects lingering in Tenured generation is low, and even lower in Permanent generation, and the probability get higher if the update rate gets lower. If the update rate is low, there are not that many immutable objects created anyway. As a result, we should have fewer major GCs and even fewer full GCs with this approach!

You might still find it counter-intuitive and still have doubts that the Garbage collectors may not efficiently processing immutable objects. Actually the contrary is true, and it was explained better [here](https://blog.takipi.com/5-tips-for-reducing-your-java-garbage-collection-overhead/). It is even better with the G1 Garbage Collector with the sensible [settings](https://www.oracle.com/technetwork/articles/java/g1gc-1984535.html).

## Don't get carried away by multi-threading

Sorry to be a pain. Having read all these, are you actually sure you need multi-threading, except to look good in your CV? If you have a single source of input, you may actually be better off applying [Mechanical Sympathy](https://mechanical-sympathy.blogspot.com/) than hoping multi-threading can magically improve performance. 

Even if you have multiple sources of input, possibly from different threads, you may be able to put a sequencer to convert it to a single source.

My honest thought is that multi-threading is a solution technique, not the root problem itself. There was never a requirement from users insisting to have multiple threads. They may like to perceive as if things are happening concurrently, and there are many solutions to achieve the same result without jumping into the multi-threading rabbit hole.

For instance, if your single-threaded application has a very low response time, the users may perceive your sequential processing as if it was concurrent. Remember, all techniques above for incremental updates have a linear sequence of Aggregate updates. 

My preference is low-latency systems is always `(Functional Programming) + (Immutability) + (Single-threaded)` but that is a topic for another day.

## A quick recap

We have covered a few techniques of read-write multi-threaded operations, and which techniques are suitable in which situation. So far we could summarise them as follows:

* Independent update - use volatile or Atomic classes
* Incremental update
  * Use synchronized block, semaphore of one permit, and Atomic classes
  * When reading is more frequent, use ReentrantReadWriteLock, potentially with double-checked locking
  * When writing is more frequent, use Immutable copy for expensive reading operations
  * An alternative approach - Immutability
    * Use Immutable objects and Persistent collections
    * Enforce version control for writing operation
    * There are no real concerns of garbage collection
    
Finally I also questioned if there was really a need for multi-threaded solutions, just to annoy you a bit.