---
categories:
- clojure
date: "2023-07-23"
description: Avoid writing brittle Clojure with tactics for flexibility and robustness.
slug: brittle-clojure-legacy-systems
tags:
- clojure
- architecture
- microservices
- brittle-clojure
- robust-clojure
title: 'Brittle Clojure: Creating Legacy Clojure Systems'
draft: false
---

This is the first in a multi-part series, "Brittle Clojure". In this series, we will consider common patterns in Clojure which yield brittle systems, as well as methods to ensure robustness.

None of the basic principles for building robust software are unique. Most literature, however, is focused on object-oriented systems. Our point of view will sometimes zoom in to Clojure "in the small", and sometimes zoom out to distributed systems built with Clojure.

Throughout these articles, we will use as a hypothetical example a legal case management system, Atticus Case Management (ACM). Its users include attorneys, clerks, accountants, and clients. It supports a variety of use cases: from hours tracking, to billing, to document creation and review, to client communications. We will use ACM as an exaggerated example of anti-patterns.

We follow the narrative of Alice, a software developer, as she joins ACM.

# ACM: A Distributed Clojure Application

Alice is excited to join ACM and start working with Clojure. There are four teams of Clojure developers, and separate QA and product.

There is a single re-frame application on the front end, and about a dozen Clojure microservices. Postgres is the data store, with Kafka as a message bus.

Ben, her team lead, takes the time to walk her through some current features in flight. They will be working on enhancements to the billing system.

Alice's task is to ensure that a case cannot be closed if there is an outstanding, unpaid bill.

Her story seems a little vague to her, so she wrote the following user story to clarify in her own mind what she was doing.

> In order to ensure that clients pay their bill, as a secretary, I cannot close a case when there is an outstanding unpaid bill.

And then she wrote:

```gherkin
Given that I am a secretary

And a case has an outstanding, unpaid bill

When I attempt to close the case

Then I am prevented from doing so

And I see a message telling me that there is an open invoice.
```

There is a microservice called "billing". Alice reads through the repository but struggles to understand it. She has no trouble identifying the handlers, database calls, but there is no single namespace that contains the business logic.

As Ben pair programs with her, she notices how easily he jumps around the code base. They start with a function that Alice was struggling to understand. Ben is able to jump through all the internal functions as though his Emacs were an extension of his mind.

Ben not only flits around the billing service code base, he opens the re-frame frontend to look at the event handlers. This is necessary to see how the billing API is being used. But, to understand the sequence of events in re-frame, he also needs to look at the "case" microservice.

Alice stops Ben, asking him if there's one place in the code base that encapsulates the business logic for her feature. "Is there one place I can go to answer the question, 'who can do what, and under what conditions?''"

Ben tells her that in order to understand the business rules, Alice will have to build up her mental model that encompasses the re-frame frontend, the billing service, and the case service.

Alice tries again. She asks if there are tests she can look at that document the business logic. Ben tells her that, while their team does write unit tests for tricky functions, in general it is too hard to test a full workflow. Their services are interconnected, and much of the coordination is in re-frame. Where there's a lot of IO, tests are not as useful, he tells Alice.

Ben continues that while he does agree that testing is important in general, on this project speed is more important than quality.

### REPL Driven Development?

As they continue pairing, she discovers that Ben's typical workflow involves clicking around in the front end application to fire the events to the backend. Even with his experience, he is not able to discern what the system does by looking at the code base. He has to manually imitate a user, and then look at the logs to observe what is happening.

Ben is quite good at using the REPL to capture and inspect values. However, Ben's REPL sessions are not saved anywhere, and once the pairing sessions are over, Alice can't easily replicate them.

Alice has the sinking feeling that the system can only be understood at runtime, by manually working with the UI and observing the events that occur across the system. Alice doesn't just have to load the whole system on her Mac, she has to load the whole thing in her brain.

### Tests

Alice intends to start by writing an acceptance test for her story. However, she can't quite figure out where to put it.

Is the billing service responsible? Or the case service? It seems to her like the service is responsible for the rules in her domain.

But the existing logic exists in the re-frame front end. There are a series of events that call various services already to see if a case can be closed. The case's status comes from the "case" services. There are some calls to services named after different types of cases ("bankruptcy", "acquisitions and mergers", etc), plus another service called "tasks".

Alice asks Ben whether the logic should go in re-frame or in the "case" service. Ben tells her it  it theoretically should go in the "case" service, but the easiest thing to do would be to put it in the frontend for now.

He tells her to use a subscription to look in the re-frame state, find the invoices, and see if any are unpaid. Then call the subscription in the view to disable the "Close Case" button.

As to tests, Ben says that there is not much "logic" because most of the code is calling other services (e.g., calling the billing service to find another invoice). All that information is already available in re-frame's app-db. This is more IO than logic, Ben says. Let's skip the tests.

Alice is a little relieved. It's her first ticket. The easier it is, the more quickly she can get it done. She has enough loaded in her brain to start to push some commits.

She's a little distracted when another developer asks Ben about some regressions he had introduced refactoring the bankruptcy service. Ben insists that the code needed to be cleaned up. Yet Alice wonders: if it doesn't work, is it really an improvement?

She pushes the thought out of her head, finishes her work, and opens her first pull request.

But as she closes her laptop for the day, she almost hears Rich Hickey's voice whispering "it was easy, but was it simple?"

# Lessons

What are the main lessons we can draw?

## The REPL is a double edged sword

The REPL is a powerful tool. It enables us the visibility needed to solve complicated problems. It also enables us to tolerate creating complicated problems for ourselves.

The dynamism of the REPL is a double edged sword. By enabling us to work with systems at runtime, it allows us to build incredibly coupled systems that can only be understood at runtime.

## The Distributed Monolith

Distributed monoliths are systems that have all the downsides of microservices and monoliths without the benefits. They are characterized by tightly coupling and low cohesion.

A common driver for the distributed monolith is the "smart" SPA, that knows too much about an application's business logic and is therefore highly coupled to backend microservices.

## Cohesive Business Logic

Take the workflow of opening a case, moving the case through its lifecycle, and closing a case. There should be one place to go that expresses that workflow in this system.

In systems where the complexity lies in the business logic, cohesion is most urgent when it comes to the business logic.

Instead of this, the Atticus Case Management system spreads the business logic between frontend and the backend services. In order to figure out who can take what action under what conditions in a workflow, the logic could be anywhere: in the view (e.g., a disabled button), in the reframe subscription, properties in re-frame's app db, in an event handler, or it could be scattered throughout the back end services.

Systems like this are incredibly brittle. They are business systems that poorly encode business rules.

## Frontends should be loosely coupled

Reframe can have a high degree of efferent coupling. Sometimes its event handling system is used to coordinate sequences of commands to services.

Reframe app databases do not support information hiding well, and in a large application, there can be a great deal of shared state. Subscriptions can reach into any path in the app db.

## Implementations that are hard to test are of poor quality

Especially in distributed systems. Clojure is no exception.

There are two difficulties with building tests. The first is not being skilled in writing good tests. Just like functional programming is a skill to be acquired, so too with testing.

The second difficulty, the more difficult one to surmount, is poor implementation. Bad implementations are hard to test.

## TDD Prevents Bad Implementations

In order to easily testable a system must be:

- Highly cohesive business logic
- Loosely coupled
- Modular (i.e., pluggable)
- Thoughtful in terms of designing interfaces
- Structured in terms of the domain
- Built to expose measurement points

And of course, these are the same properties that make robust systems.

Writing the tests first makes poor implementation design very difficult.

Remember that Alice could find no one place where that encoded the business logic for a particular workflow. If Ben's team used test driven development, they would have started with high level tests expressing what the workflow does. This would have forced them to have a clear, unified place that expressed the business logic.

As a result, it would not be possible, or at least it would be extremely difficult, to build a system that has a low cohesion with the business logic.

Consider the alternative: a team writes an implementation and then considers how to write a test for it. They come to the conclusion that the test is hard to write. (It's a problem with testing itself, not our implementation.) It's not even clear which repository the test belongs in!

They have discovered their implementation to be poorly designed. But rather than think critically about their design decisions, they are so committed to their prior thinking that they give up on testing at all!

This is how to write a legacy system.

### Is TDD a must?

I don't think the research is conclusive enough (yet) to say that test driven development is a professional duty of care. However, writing thorough tests [is](https://dora.dev/devops-capabilities/technical/test-automation/). Just as an attorney keeps a client's confidence, and a doctor prescribes a certain standard of care, developers owe their employers and coworkers tests and testable systems.

And test driven development makes it much easier to Design Systems to be testable, and write the test.

## Writing testable systems is mostly a culture change

The benefits of writing tests are hard to grasp unless one i) already has a good understanding of software design, and ii) one has empathy for other developers.

Developers who understand the importance of low coupling and high cohesion, who insist on distinguishing the "easy" from the "simple", who think carefully about domains and their boundaries, tend to get the value proposition of tests.

Appreciating tests means changing core beliefs about what makes good software.

## There is no tradeoff between speed and quality

One common canard in software development is the proclamation that there is a tradeoff one must between speed and quality, with the implication that tests slow you down.

In fact, empirical research has shown this to be false: speed and quality are positively correlated. See Forsgren, Humble, and Kim's _Accelerate_. And writing tests is a driver of both.

The idea that there is a tradeoff comes from a blinkered view of the span between receiving a ticket and tossing it over the wall. Viewing the system as a whole (realizing the cost of defects, for example) and having empathy for other developers who must work with the code requires a broader perspective. It requires an understanding that the mental model you build up when writing the code is undocumented, not reflected in any executable specification.

Taking a broader perspective, and having the rationality to submit our anecdotal opinions to the evidence is not easy. Like [Cremononi refusing to look through the telescope](https://thomascothran.tech/2023/07/dont-be-cremonini/), many will simply ignore evidence where it discredits their own opinion.

## Writing Legacy Clojure

Michael Feathers defined legacy systems as those without tests. This is precisely the problem Alice faces: her new team's code base is a legacy system from the jump.

Righting the ship is going to require a dual approach. It requires two partitions. For new features, a test driven approach will go a long way toward a simpler, robust, understandable system. For the untested parts currently in place, the team will need to use strategies for managing legacy systems.
