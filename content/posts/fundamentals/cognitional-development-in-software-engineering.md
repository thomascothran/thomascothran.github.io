---
categories:
- Fundamentals
date: "2023-11-24"
slug: cognitional-development-for-software
tags:
- engineering
- cognitive psychology
title: Cognitive Development for Software Engineers
draft: true
---

In Lawrence Kohlberg's famous Heinz dilemma, participants are told a story about a man who steals medicine to save his wife's life and are asked to judge his actions. Children at lower stages of moral development (preconventional level) focus on concrete individual consequences (e.g., Heinz will get punished) and cannot comprehend the moral significance of life preservation that participants at higher stages (conventional and postconventional levels) understand, where societal rules or ethical principles are considered.

The psychological notion of levels of cognitive development can be very useful in explaining why software developers have starkly different views on the fundamentals of software design. Just as those at a lower stage of moral development cannot conceive the higher, more abstract principles of values and responsiblities, so those at a lower stage in the context of software engineering struggle to understand higher level principles of software design.

A distinction favored by the philosopher Wilfred Sellars is useful here. There is what one sees *of* something, and what one sees that something *as*. A physicist and a non-physicist may both look at an image of an electron cloud. What they see of that electron cloud is the same: blue specks, becoming more and more dense toward the center of the image. Yet what they see the image *as* varies dramatically. The non-physicist may have a foggy memory of electrons from grade school (something about dimensionless points and charges), or even have the view of an educated layperson. Yet what the physicist sees is quite different, because of the mastery of the discipline of physics.

So it is with software development. What we see of a code base, a function, a pattern, or a principle may be the same. Yet what we see it *as* can vary dramatically.

Paul Graham's "Blub Paradox" illustrates the problem. Graham postulates a language, "Blub", that sits in the middle of the abstractness spectrum.

> As long as our hypothetical Blub programmer is looking down the power continuum, he knows he's looking down. Languages less powerful than Blub are obviously less powerful, because they're missing some feature he's used to. But when our hypothetical Blub programmer looks in the other direction, up the power continuum, he doesn't realize he's looking up. What he sees are merely weird languages. He probably considers them about equivalent in power to Blub, but with all this other hairy stuff thrown in as well. Blub is good enough for him, because he thinks in Blub.

Graham is defending abstract programming languages. Our question concerns the capacity a programmer has for abstraction. But the same dynamic is at work in both cases.

Consider an example. A group of programmers attends a lunch and learn about the importance of cohesion and techniques to reduce coupling. The reactions to the talk are varied. Some immediately grasp the meaning and significance of the core concepts. Others fail to see the value in modular systems, perceiving the core principles as "academic" abstractions and the techniques as doing things "the long way".

We can explain these spontaneous differences in terms of a three-tiered model of cognitive development. Each lower level is a pre-condition for the higher level. There are three levels: Level 1 (Fundamental Implementation), Level 2 (Problem-Solution Distinction), and Level 3 (Strategic Problem-Solving). Although I will use terms like "the level 1 developer", this is shorthand for "a developer operating on level 1". The same person can operate on different levels depending on the circumstances.

# Level 1: Pre-Problematic

At level 1, software engineers grasp the syntax of their programming language, and can turn requirements into working software. Their focus is on implementation, on getting things working. Understanding software means being able to follow its execution steps. 

For example, the level 1 programmer would understand a web service if they could follow how a response is generated from a request. Given the structure of a request, they can identify the data sources from which further information is gathered, follow the transformations of the request and the further information, see the updates that are made to the application's state, and so on. 

The ability to operate at level 1 is necessary to further develop one's engineering abilities. But operating only on that level leads to poor quality software.

Consider two personality types that operate at level 1: the interpreter and the tactical tornado.

## The Interpreter

Developers operate as interpreters when they view their work as "translating" requirements into software, much as a classicist might translate Homer's Iliad from Greek to English. The interpreter presumes the product owner or business analyst knows perfectly well what they need. It's just a matter of translating these requirements to the language a computer can execute.

This often manifests as "programming by remote control". In extreme examples, the product owner or business analyst will outline application logic in pseudocode or dictate the structure of database tables.

## The Tactical Tornado

John Osterhaut characterizes another type of developer that operates only at level 1: 

> The tactical tornado is a prolific programmer who pumps out code far faster than others but works in a totally tactical fashion. When it comes to implementing a quick feature, nobody gets it done faster than the tactical tornado. In some organizations, management treats tactical tornadoes as heroes. However, tactical tornadoes leave behind a wake of destruction. They are rarely considered heroes by the engineers who must work with their code in the future. Typically, other engineers must clean up the messes left behind by the tactical tornado, which makes it appear that those engineers (who are the real heroes) are making slower progress than the tactical tornado.

The tactical tornado is "tactical" as opposed to "strategic". That is, their focus is on a working implementation rather than good software design.

## Other Characteristics

The interpreter and the tactical tornado are two personality types. There are an indefinite number of others. They share many of these characteristics:

- Abstractions and generic operations are viewed as "complex"
- Low level concerns are present in high level components (e.g., domain logic on your backend is influenced by how JSON represents data)
- When developing, drives the application manually (e.g., by clicking in a UI or firing HTTP requests to a running backend) rather than programmatically (e.g., from a test or rich comment block)
- Tends to "lock on" to a single approach, rather than considering alternatives.
- Views "readable" code as code that organizes implementation details so that the flow of execution is easy to understand

These similar characteristics all follow from identifiable limitations in understanding software engineering. These limitations will be clearer as we consider the next level.

# Level 2: Problematic

The limitations of the level 1 developer point toward the second level. The level 1 developer thinks in terms of implementation. The level 2 developer thinks more abstractly.

The core skill of the level 2 developer is how they refine the problem space and distinguish it from the solution space. A programmer stuck at level 1 thinks of the "problem" to be solved as how to implement requirements. The level 2 engineer realizes that problems take work to discern. Like a medical doctor, they do not confuse symptoms with a diagnosis. Problems are not given ready-made, they are produced by careful thought and the collection of relevant data.

We learn this in grade school mathematics with word problems. Facility in mathematics is not about being able to execute a series of steps correctly. It is being able to move from the presentation of a problem to its formulation. It's what happens *prior* to mechanical computed. Similarly, writing working code is relatively easy. Producing good solutions that solve a well-defined problem is harder.

For example, if a product owner asks for a table of data, the level 1 developer will take the "problem" to be about how to build the table. If they were operating on level 2, they would try to get to what the end user is ultimately trying to accomplish, and what the business value is.

The level 2 programmer aims to clearly articulate the nature of the problem, and strictly distinguishes it from the solution space (features that would solve the problem). After formulating the problem, the level 2 programmer considers alternative solutions. 

Level 1 programmers can masquarade as level 2 engineers. A few giveaways:

- Solutions (implementations) tend to precede clear problem statements
- The process of thinking through the problem rarely changes the chosen solution

## Understanding Software

The level 2 engineer understands software differently than at level 1. There are different cognitive skills at work. 

Because they are operating at the problem-solution level, they are more concerned with the "what" than the "how". Understanding code means grasping its role in the business context, not following its execution.

This has extensive and profound effects on how software is written and read. The level 2 engineer disagrees with the level 1 view that software is readable if it clearly expresses "how it works". Rather, the level 2 engineer recognizes that implementation details are inessential; what is essential is the meaning. 

As a result, the software that level 2 engineers write will hide implementation details behind interfaces. It will seek to clearly express the domain. It will be compartmentalized so that it has limited, specific places of coupling. The level 2 engineer focuses on building good abstractions, without excessive emotional attachment to implementations.

The term "interface" is a general one. In Clojure we have a number of interfaces we can work with: HTTP APIs, functions, function composition, multimethods and protocols. The important point is not the tool, it's the purpose: separating "what" from "how".

## The Chasm

Between the engineer operating at level 2 and the developer at level 1, there is a chasm. Engineers can look at the same code, hear the same words, use the same terms. But the frame in which these are understood differs radically.

The difference in framing stands at the root of many of the differences on the observable level. The level 2 developer will seek to reduce coupling; the level 1 developer thinks its easier for everything to have access to everthing else. The level 1 developer will see information hiding as "hiding the ball"; the level 2 developer will see it as a means of separating the essential from the inessential and reducing complexity. For example, the level 2 developer will grasp immediately why microservices should generally not reach into another microservice's database. The level 1 developer will merely view this as a hindrence.

If the level 2 developer tries to persuade the level 1 developer of the reasons why this ought to be avoided, they are unlikely to get far. This is not because of the merits of the argument, but rather the cognitive capacities each has acquired. The world of the level 1 developer is limited to the world of implementation details, of concretions. From that perspective, reducing coupling between services or across layers, or controlling them through interfaces, merely gets in the way.

The level 2 developer understands implementation details, but theirs is a broader, more comprehensive point of view.

# Level 3: Meta-Problematic

The level 2 developer is able to consider particular problems and design good solutions. At level 3, a further problem emerges: how to consistently and regularly solve problems well, given incomplete knowledge. Were it the case that we always have all the relevant information, there would be no need for a third level.

In the real world, complete knowledge is usually a fantasy. Rarely do we get things right at first. Our knowledge develops through time: at the outset of a project, we know less than when we are half-way through. We learn more about the business domain, the interaction of different services, etc. as we move along.

Our knowledge is also, as it were, "spatially" constrained. We work in complex systems, systems which are too complex to understand all at once. A system that entirely "fits in one's head" is too trivial to provide a real business advantage.

More fundamentally, complex systems do not *in principle* reduce to a single unified understanding. A system is complex precisely because it does not reduce to a single, consistent model. It is exemplified not by things like a solar system, but the weather. It is easy to say precisely where Mercury will be one year from now. It is very difficult to say with any accuracy what the weather will be in New York city.

The key insight in the transition from level 2 to level 3 is the recognition of the problem of partial knowledge. But if we cannot envision all the problems that we need to solve, what use is it to try to do anything about it? An example will be useful. 

Consider Greg Young's advice in his talk, "The Art of Destroying Software". Young recommends writing code so that it can be deleted and re-written in no more than a week. The effect of writing software this way is that risk is dramatically reduced: no matter what mistakes are made, the cost of fixing them is limited to a week's worth of work.

Young is operating on the third level. The problem he is dealing with is not solving a particular business problem. It's the problem of how best to solve problems. And he is advocating that, in solving business problems, we should minimize the risk intrinsic to our solutions.

Level 3 operates on the level of principles. Principles like the open-closed principle apply across different types of problems and domains. This is not to say that principles need to be applied mechanically. However, if there are tradeoffs in the small, nevertheless principles must guide a service or project in the large. For example, one may decide in some particular case that a high level policy may depend on lower level details; but one operating on level 3 will, for the most part, produce software where the high level policies are independent of low-level detail. They will not move from the view that there are exceptions to the belief that they are always the exception.

## The Chasm Widens

The gap between the level 1 programmer and the engineer operating on level 3 is huge. While the level 1 programmer regards level 2 ideas as complex, at least they have some connection (however academic) to the software we write. But asking the level 1 developer to anticipate problems that haven't arisen yet is simply baffling. It lies so far beyond their horizon as to be almost meaningless.

By contrast, the path from level 2 to level 3 is much easier. There are real differences. For example, overengineering occurs on Level 2, when one overestimates ones ability to anticipate future scenarios. But the real challenge to maturation as a software engineer is in the transition from level 1 to level 2. Once the skill of thinking abstractly, of systematically solving problems without being hung up on implementation details is achieved, the path from level 2 to 3 is a natural one.
