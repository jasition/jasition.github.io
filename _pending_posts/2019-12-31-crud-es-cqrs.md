---
layout: post
title: CRUD, Event-sourcing and CQRS
cover: cover.png
date:   2019-12-31 13:00:00
categories: posts
---

# Demigods and the almighty
To the software programs, we the programmers are the demigods, nearly the almighty. We decided how the little world inside a program worked. We made rules and we controlled how things were transformed in this program world.

However, we still relied on the programming language, the operating system, cables, chips, and electronics behind the scene. Therefore we are quite divine, but not almighty.

Without the ability to defy the law of physics, we have a lot of leeway in a program to navigate in order to automate processes and solve problems.

# Let's talk about application semantics
We definitely has the power to define the meanings of data structures and functions. Here we do not necessarily refer to the business meaning of them. It can do, but not under of the scope of the discussion here. We could define the functional meaning of different constructs or syntax  within a program. A more fancy term is the *"Application Semantics"*.  

## A simple example
I can give an example about Application Semantics.

Given there is a REST endpoint that handles the request of saving a number. The request body is a number. A success response returns HTTP status code 204, implying it was a success, i.e. the number is saved, but there is no content in the response body. An error in the request can be HTTP status code 406 for the number that cannot be accepted, or 413 if the number is too big, etc.

Those HTTP status codes, request body and response body are the syntactic constructs. "No content (Saved)", "Not accepted", "Payload too large" are the semantics of what the syntactic constructs actually mean. A human interprets HTTP status code 204 in this case as, "Oh my number is now saved". 

# Create, Read, Update, Delete (CRUD)
This is the classic Application Semantics that put operations into four categories.

| Operation  | SQL Command  | REST  |  
|---|---|---|
| Create  | Insert  | POST  |  
| Read  | Select  | GET  |  
| Update  | Update  | PUT  |
| Delete  | Delete  | DELETE  |

They are very clearly defined for what they mean from their literal names. CRUD implies that each resource can be uniquely identified, in other words, each resource is keyed on something. The key is used in all operations, so that for example, a Delete operation can be requested by just the key.

## The current state is all that matters
Each resource being keyed by unique identifier implies that there is always a latest state of the resource. The historical states are replaced and forgotten. There is no duplicate resource by definition. Of course you could spawn a history item for the audit trail, but that resource is a resource of history, but the original resource itself. This is in short the "Start of the World" semantics.

CRUD is best used in the problem where the current state is the only thing that matters. The old states are quickly disposed and it is easy to get the latest states. In the financial world, market data distribution is a the classic use of CRUD. 

The market data client subscribes the market data identified by the symbol of the financial instrument. The first response is the current state of the item requested. Any update or deletion are pushed to the client as the client subscribed to the item. Also the symbol list or security list informs the client the creation of a new item. Live trading cares only the latest market data to the point that any old market data are discarded using conflation.

In one sentence, CRUD is the semantics of "What is the current state?".

# Event Sourcing
On the other hand, Event Sourcing sees the world has an incremental sequence of events that shapes the state of the world continuously. There are numerous versions of the state of resource, and applying all events up to the latest event equals to the current state.

## History and Traceability
All events are kept forever and you can go back in time to see a historical state. The resource can still be keyed, and if it does, each event only concerns one resource, and events of that resource becomes effectively the stream of events. Creation is equivalent to the start of stream; Update is just an event in the stream; Deletion is the end of stream; and Read is simply the traversal of the stream.

Practically the program always keep the latest state by playing all events until the latest, so that it could compute the next state faster. Interestingly, the current computed state could actually be read and the program afterwards can generate update or delete request that result in an event appended to the stream.

Event Sourcing is best used in the problem where history matters and there is need to trace the causality of state changes. Using an example in Financial world, anything that needs to operates with the current state and required to have audit trails would be an excellent fit, such as a book of client orders. When queries by authorities or regulators, we could re-play event by event and work out how we ended up with the current state. Just like debugging our code line by line, Event Sourcing can be used to debug the state change event by event. 

In one sentence, Event Sourcing is the semantics of "How did it get to the current state?".

## Define the right events
* It is business meanings to downstream system


# Event-sourcing

# Reactive Programming

# Modelling the reality

