---
layout: post
title: Write Tests for Conceptualisation 
cover: cover.png
date:   2018-11-01 00:00:00
categories: posts
---

## Should I even bother?

*"I wrote tests for each method", "My code coverage is 100%", "All my tests passed"* - So why should I even bother? It does not matter which level you are testing in the [Testing Pyramid](http://www.duncannisbet.co.uk/test-automation-basics-levels-pyramids-quadrants), you probably will have heard these *evidences* that attempted to prove they were good tests. 

What makes a test good though? A test can only be good if both the authors and the readers agree. You probably realise by now that those *evidences* mentioned above all came from the authors only. How do we, as readers, determine whether a test is good, given we did not know what it tests? 

Just like everything else in the world, as the readers, the *consumers* of the material, we can be demanding. We may ask these questions:

* What does it test?
* What happens if (input is invalid / remote service not available / resource not found ... )?
* What is it we are testing, really?

Good tests should be able to give readers the answers, without looking at the source code.

Tests are everything we could understand of a function from an external point of view. The test suite is the user manual. Tests define the function under test.  

## How does a test defines a Java class under test?

This question is similar to *"What defines a chair?"*. You could probably come up with these definitions:

* Has 4 legs typically
* Has a back for one person
* Often has rests for the arms
* Supports the person's weight When sat on 

Inversely, if you described the 4 above points to people and ask them what the item could be, some of them will answer "Chair". And yes, this is how we know it was a Chair even we have not even seen it ourselves.    

Some of the definitions are the properties and some of them are behaviours. If Chair is a Java class, the unit test should test its properties and behaviours, in order to convince the readers that this is a Chair.

```java
public class ChairTest {
    public void has4Legs();
    public void hasABackForOnePerson();
    public void hasArmRests(); // If it is supported
    public void supportsPersonWeightWhenSatOn();
}
```

Every test method is a definition of the class under test, and the test names are in plain English that everyone understands. It is almost a direct translation.

Now you should understand why a test method name like *testThisMethod* means absolutely nothing. There are some conventions that are commonly used:

* adjective : properties, attributes, and immutable behaviours.
* when-then : reactive behaviours based on parameters and/or initial states.
* given-when-then : reactive behaviours based on parameters and/or given states and/or pre-conditions.

## Getting personal

If you are going to buy a chair for yourself, you want to know this chair more. You may want to know if the chair

* is wide enough for you
* can support your body weight
* is comfortable to sit on
* can last for years
* is easy to fix when damaged

then they all get personal. And they should all be translated to your test cases. 

* (edge case) is wide enough for you, with your hip width as input parameter 
* (edge case) can support your body weight, with your body weight as input parameter
* (usability) is comfortable to sit on
* (load test) can last for years
* (resilience and recovery test) is easy to fix when damaged

You know this chair is good enough for your money if **all tests have passed** and **the test suite has good coverage**. Why would software be any different?

## Excuse me, usability? I am testing my Java class only

Yes, I hear you. Usability does not just mean Graphical User Interface. If you want me to use your Java class, I need to know how easy it is for me to use it in my code. There is no better way to show me than demonstrating it in your test cases. 

This is the cliché of *"Eat your own dog food"* that if your test cases use your own class in an awkward way, what chance do the users have?

Your test cases are the opportunities to make your API shine. It demonstrates how fluent and how usable is your API. **Your test suite is a user manual and it put your API on show.**

Here are two examples of test cases that verify the same behaviour, but the latter has considered the usability and demonstrated it in the test case. 

Example 1:

```java
   public void testFuse() {
      fuser.add("Hydrogen", 2);
      fuser.add("Oxygen", 1);
      
      fuser.run();
      
      String actual = fuser.getElement();
      
      Assert.assertEquals("Water", actual);
   }
``` 

Example 2:

```java
   public void fuseH2OBecomesWater() {
       assertThat(fuser
                  .givenElement("Hydrogen", 2)
                  .givenElement("Oxygen")
                  .fuse()
                  .getResult()
                 ).isEqualsTo("Water");
       )
   }
``` 

The usability test is sometimes subjective and it is not easy to define criteria that are repeatable and reliable. There are experiment methodologies in Psychology to help with it (e.g. short-term memory, motor memory tests), but unfortunately it cannot not automated yet, as it involve reactions from human.  

## Write tests for conceptualisation

Tests could allow the readers to conceptualise the function under test. A good test suite should be able to

* Define the properties
* Define the behaviours
* Define the limits
* Demonstrate that it has fulfilled the given non-functional requirements
* Demonstrate the usage

to the extent that they will use it. Be it Java Class, Component, Module, API, Application, System or a collection of Systems.

Write tests for conceptualisation. Write tests to convince yourself that you are happy to use your own APIs.