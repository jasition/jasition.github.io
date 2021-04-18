---
layout: post
title: Search for the perfect DSL
cover: cover.png
date:   2021-04-18 13:00:00
categories: posts
---

# I forgot what I have done and why

I was troubled lately when I looked back to the project I worked on before, that I found it difficult to understand what was going on. Funny enough, I wrote the code myself and I had no recollection what I supposed to do.

I forgot what I have done and why. Sound familiar? How could one forget something one worked hard on before?

Yes we do forget, and in my case, very often. I had to understand the behaviour of my code by reading my *extensive* suite of test cases. I managed to understand what the test was trying to do, but I still struggled to understand why. 

And yes, I think this was the problem. *I understood what the code was doing, but I did not know the purpose of it*. My old code was not expressive enough for someone as forgetful as me to read and revise.

## Is your code expressive enough?

When I wrote BDD tests in Kotlin, I used the approach of data-driven testing from [Kotest](https://kotest.io/docs/framework/data-driven-testing.html) to write as many test cases as I can, knowing that I can parameterise to do more with less effort. 

I think I was half-correct. I ran into the traps where I struggle to understand what was going on.

For example, this is the actual I took from my personal project [Matching](https://github.com/jasition/matching) which is about order book matching.

```kotlin
    forAll(
        /**
         * 1. Passive  : bid size, bid price, offer size, offer price
         * 2. Aggressor: side, type, time in force, size, price
         * 3. Trade    : passive entry index (even = buy(0, 2), odd = sell(1, 3)), size, price,
         *               aggressor status, aggressor available size, aggressor traded size,
         *               passive status, passive available size
         *
         * Parameter dimensions
         * 1. Buy / Sell of aggressor order
         * 2. Single / Multiple fills
         * 3. Exact / Better price executions (embedded in multiple fill cases)
         * 4. Stop matching when prices do not match (embedded in single fill cases)
         */

        row(
            List.of(Tuple4(6, 11L, 6, 12L), Tuple4(7, 10L, 7, 13L)),
            Tuple5(BUY, LIMIT, IMMEDIATE_OR_CANCEL, 16, 12L),
            List.of(
                Tuple8(1, 6, 12L, PARTIAL_FILL, 10, 6, FILLED, 0)
            )
        ),
        row(
            List.of(Tuple4(6, 11L, 6, 12L), Tuple4(7, 10L, 7, 13L)),
            Tuple5(SELL, LIMIT, IMMEDIATE_OR_CANCEL, 16, 11L),
            List.of(
                Tuple8(0, 6, 11L, PARTIAL_FILL, 10, 6, FILLED, 0)
            )
        ),
```
I know I was leveraging parameterisation to test many scenarios with less code, but there were several problems:
1. I have no idea what the hell is going on. How might we add a new test?
2. How does it define or [conceptualise](2018-11-01-tests-for-conceptualisation.md) the behaviour of the program?
3. Why are we testing it?

The large comment section was *trying* to help only to confuse myself even more. And I was also facing the question whether I could trust that the comment was up-to-date. I had to double check the code and verify if the comment was right.

There was a lot of time wasted in this. There must be a better way.

On the one hand I can train my memory to be better, but wait...if it was a team repository then it will not help. I need to improve the readability of the code.

# How about the existing design patterns?

This is certainly not a new problem, and for that reason, there must be some design patterns around to tackle the problem. Let's have a brief look at the popular ones and see if they improve it.

## Arrange-act-assert (AAA)

[Arrange-act-assert (AAA)](https://freecontent.manning.com/making-better-unit-tests-part-1-the-aaa-pattern/) is a common pattern, especially when it comes to unit testing.

The first part is the arrangement, which sets up the scene for testing.

The second part is the execution of the production code we intend to test.

The final part is the verification of the results.

Sounds pretty simple huh? Yes and it is useful too. An example of the AAA patern can look like this:

```kotlin
@Test
fun canEnterTheShopIfIWearAMask() {
    // arrange
    val customer = Customer.withAMask()
    
    // act
    val result = shop.checkWithSecurityGuard(customer)
    
    // assert
    result shouldBe Result.ALLOWED
}

```

## Given-when-then (GWT)

[Given-when-then (GWT)](https://en.wikipedia.org/wiki/Given-When-Then) is another popular style which is commonly used in Behaviour-driven testing (BDD).

You can almost map these 3 parts to those from the AAA pattern we just went through:

| AAA     | GWT   |           |
|:--------|------:|----------:|
| Arrange | Given | Pre-test  |
| Act     | When  | Test      |
| Assert  | Then  | Post-Test |

However, *Given-when-then* is closer to natural English, and is easier to undestand especially non-technical stakeholders.

Writing it in the [Cucumber](https://cucumber.io/) feature file may look like this:

```

  Scenario: Entering a shop with face mask on 
    Given I wear a mask
    When I check with the security guard
    Then I should be allowed to enter the shop

```

The natural English is very expressive and easy to understand. However, there is a small cost in connecting the natual English to executable code. See below:

```java
@Given("I wear a mask")
public void I_wear_a_mask() {
    // Write code here that turns the phrase above into concrete actions
    throw new io.cucumber.java.PendingException();
}

@When("I check with the security guard")
public void i_check_with_security_guard() {
    // Write code here that turns the phrase above into concrete actions
    throw new io.cucumber.java.PendingException();
}

@Then("I should be allowed to enter the shop")
public void i_should_be_to_enter_the_shop() {
    // Write code here that turns the phrase above into concrete actions
    throw new io.cucumber.java.PendingException();
}
```

# Search for perfection

I, of course at the end, managed to understand my code and moved on. But I was not happy that I had spend all that time. I needed a better way to express in the code so the reader need not spend hours cracking the code just to understand how it supposes to behave. 

Did you ever keep searching for perfection? and the worst part, you did not find anyting satisfying the one thing you want. This one thing to me is expressive test code that a domain expert can easily understand.  

I found both Arrange-act-assert and Given-when-then useful, but there are quite some bits something to achieve my goal. 

So I started a new direction by creating DSL (Domain-specific language).

# How does DSL (Domain-specifc Language) come to the rescue?

Actually, the natural English in the feature file above is already DSL. It is natural enough for a domain expert to comprehend. It uses the ubiquitous language applicable to the domain.

So why I am still not satisfied by the BDD framework like Cucumber? I think it is the need to write the test fixtures that somehow is a little awkward for me.

Ideally I want the DSL itself to be verified at compile-time. I want it to be readable at source level. No translation is required as fixtures.

Wait, I think [Spring MVC](https://spring.io/guides/gs/testing-web/) is already doing it:

```java
    @Test
    public void return415ForUnsupportedMediaType() throws Exception {
        mvc.perform(put("/transactions/transaction/" + UUID.randomUUID())
                .contentType(MediaType.TEXT_PLAIN)
                .content(DtoToJson.convert(getPendingTransactionDto()))
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isUnsupportedMediaType());
    }
```

It turns out I can have the best of both worlds. I don't need to translate natural English to code, while keeping the code more of less like natural English. Of course at a small cost of noisy characters of dots and brackets.

# That's not perfect, is it?

No it is not perfect, but it is close enough. By 

1. Close to natural English. That means it is easy to understand and expressive. 
2. Compile-time safe. You need not worry about how the DSL magically becomes executable code.
3. Your IDE auto-completes the syntax for you.
4. It is very extensible and re-usable. You can compose complex test cases by building the elements together.

# An example how to build your own DSL for testing

Building your own testing DSL sounds like a daunting task. It may seem a bit overwhelming to start. However, with a bit of my experience shared here, you can divide and conquer it.

I have already foreshadowed that there are three parts of a test. Same should apply to DSL testing:

1. Arrange (Given). Create a builder class to collect information which scenario you intend to build. For readability, you might want to create some syntactic sugars too. The most important output here is the pre-test state. Please note that each function return the test case builder itself to allow adding more building blocks to the test.

```kotlin
    fun given() = this
    fun `when`() = this
    fun and() = this
    fun then() = this

    fun hasAMaskOn(maskOn: Boolean) = apply { this.maskOn = maskOn }
```

2. Act (When). The actual code we want to test given a pre-test state. It can either direct act on the pre-test state, or suspend it until you need to verify. Make sure you capture the result for assertions later. 

```kotlin
    fun checkWithSecurityGuard() = apply { this.result = securityCheck(this.customer) }
```

3. Assert (Then). The expected result is passed in and verified. 

```kotlin
    fun expectAllowedToEnterShop(expectedResult: Result) = apply {
        result shouldBe expectedResult
    }
```

# Putting them all together

I have an example when I fully apply the DSL in my BDD tests. The test class itself is the one-test summary of the behaviour, and the feature value `aggressor as buy order` is the variant or context of the test.

Please note that I suspended the execution by passing the lambdas when building the test case. The test is only built, run and verified when the function `arrange`, `act` and `assert` were invoke respectively.

```kotlin
internal class `partially filled aggressor is cancelled and no book change` : FeatureSpec({
    val david = defaultBrokerClient()
    val karen = defaultBrokerClient(firmClientId = "Karen")

    feature("aggressor as buy order") {
        givenABook()
            .isEmpty()
            .`when`().placingAnOrder {
                it.side(BUY).size(8).price(12).timeInForce(GOOD_TILL_CANCEL)
                    .requestedBy(karen).requestId(ClientRequestId("buyOrder"))
            }
            .then(startWithEventId = 1)
            .expectAnOrderPlaced {
                it.side(BUY).available(8).price(12).timeInForce(GOOD_TILL_CANCEL).status(NEW)
                    .requestedBy(karen).requestId(ClientRequestId("buyOrder"))
            }
            .and().expectAnEntryAddedToTheBook {
                it.side(BUY).available(8).price(12).timeInForce(GOOD_TILL_CANCEL)
                    .status(NEW).requestedBy(karen).requestId(ClientRequestId("buyOrder"))
                    .entryEventId(1)
            }
            .and().expectAnEntryPrioritisedInTheBook {
                it.side(BUY).available(8).price(12).timeInForce(GOOD_TILL_CANCEL)
                    .status(NEW).requestedBy(karen).requestId(ClientRequestId("buyOrder"))
                    .entryEventId(1)
            }
            .arrange()
            .act()
            .assert()
    }
```

My project can be found [here](https://github.com/jasition/matching).