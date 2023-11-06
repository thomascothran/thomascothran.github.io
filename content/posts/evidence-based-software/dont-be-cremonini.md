---
categories:
- Evidence-Based-Software
date: "2023-07-09"
description: Are debates about software just differences of opinion? Or do we have the evidence necessary to identify what works and what doesn't? How the Annual Devops Report can resolve questions about the best practices in software development
slug: dont-be-a-cremonini
tags:
- DORA
- empiricism
- evidence-based-software
title: Don't Be a Cremonini
draft: false
---

Differences of opinion about how we ought to write software have an air of the philosophical about them. Some prefer TDD and microservices, others may prefer monoliths and think that most testing is a waste of time. Or engineers may prefer to use continuous development methodologies, while businesses prefer a waterfall approach with decorative scrum ceremonies.

Are we stuck with opinions? Must we be subjected to the obligatory "well, in my experience ..."? Are we simply expressing our personal feelings, or is there some truth to be had?

Fortunately, we have the tools to get past opinion and subjective experience. Just as we no longer hold that the world is flat, that the earth is the center of the universe, or that planets move about in crystalline spheres, so there are opinions in the context of software development that are no longer intellectually respectable to hold.

# Aristotle and Galileo

Our situation is much like early modern science.

For centuries, people thought that things fell faster in proportion to their weight. They thought the planets were unchangeable. For this they had a lifetime of anecdotal evidence and the authority of an expert: Aristotle.

Galileo’s revolution lay in submitting the experience accumulated over millennia to rigorous quantitative measurement.Not everyone was convinced right away, Galileo recounts to Kepler that many Aristotelian philosophers refused even to look through his telescope. His colleague Cesare Cremonini allegedly even said:

> I do not wish to approve of claims about which I do not have any knowledge, and about things which I have not seen … and then to observe through those glasses gives me a headache. Enough! I do not want to hear anything more about this.

The lesson? Even with the evidence, one can only make progress if one is willing to abandon one’s own opinions in light of the evidence. Don’t be a Cremonini!

# Empiricism > Intuition

Is pure science is a special case? Does quantitative hypothesis testing does work in complex, concrete, _human_ scenarios? Surely outside carefully controlled laboratories, our intuition and experience is more reliable than abstract, quantitative techniques.

Evidence based medicine provides an instructive example. Its great struggle arose not so much from technological factors as human resistance. Doctors insisted that their intuition was more accurate in particular cases than the results of large studies.

And we see why this is so psychologically difficult: what is the value of long years of experience if it can be set aside for a table of numbers?

Now of course we are fortunate that when we need medical care that evidence-based medicine has been institutionalized as the standard of care

Nor is the medical case unique. Academics such as Paul Meehl and Philip Tetlock have studied for decades whether expert intuition is more accurate than very simple empirical statistical models in a variety of domains: psychology, geo-politics, economics, criminal justice, and other social sciences.

Which performs better? Intuitive judgments which rely on expert experience? Or quantitative empirical methods? The weight of the evidence is clear: quantitative-empirical methods are superior both for predicting general trends, and accurate judgments in particular cases.

This should not surprise us much. Consider how difficult it is to prioritize development projects without a financial analysis that predicts how each project affects profits and losses. Prioritization not based on hard numbers are inevitably based on how important something "feels".

Again, consider how difficult estimation is when we try to use our intuition to determine how long a project might take. By comparison, statistical techniques like reference based forecasting or monte carlo simulations not only are faster, but more accurate.

# Empirical Evidence for Software Development

The [Annual State of Devops report](https://dora.dev) is the largest empirical study of the effectiveness of software development practices. Not only does it look at technical measurements (such as lead times); it considers economic effects such as profitability and market share.

Dora has gathered data from over 33,000 software developers over the last decade. The findings are summarized in a book called Accelerate.

Perhaps the best way into the DORA findings is the new [capabilities](https://dora.dev/devops-capabilities/) page. It sets out those practices which have empirical support.

In addition to technical capabilities (such as [Cloud Infrastructure](https://dora.dev/devops-capabilities/technical/cloud-infrastructure/) and [Source Control](https://dora.dev/devops-capabilities/technical/version-control/)), there are process and cultural capabilities.

# Beyond opinion

Consider some common debates developers have. Most developers accept testing as part of being a software professional, yet there are many dissenters. Even those who favor testing often say that testing involves a tradeoff of speed for reliability.

But what does the evidence say? It may be unsurprising to learn that the conventional wisdeom is correct -- testing does improve reliability. What is more surprising is that _there is no tradeoff between speed and reliability_!

The effect is that the debates developers have have, at work or on Hacker News, around [testing](https://dora.dev/devops-capabilities/technical/test-automation/) or [continuous delivery](https://dora.dev/devops-capabilities/technical/continuous-delivery/), can be resolved. Everyone has opinions of course. The question is whether they are intelligent, informed opinions.

And there are many things we can rule out as discredited (until more comprehensive evidence appears to the contrary). Here are a few of my favorites:

- Tests don't matter, or they slow you down, or developers don't need to be involved in writing them
- Speed and reliability are opposed to one another
- It is safer to deploy weekly or monthly than daily or hourly
- Gitflow is better than trunk based development
- Batching features is better than releasing incrementally
- Once written (or pulled into a sprint), requirements should not change
- Microservices will slow you down
- Tight coupling is preferable to loose coupling

Prior to the availability of empirical evidence, we are all Aristotelians, doing the best we can with our common sense experiences. But after the evidence is available, asserting one's common sense is no more intelligent than Cremonini's refusal to look through the telescope.

# Engineering and the Broader Organization

Often, though, engineers are not debating among themselves, they are negotiating with business analysts, quality assurance departments, security teams, product owners, and managers.

For example, engineers are more liable to want [test](https://dora.dev/devops-capabilities/technical/test-automation/) and [deployment automation](https://dora.dev/devops-capabilities/technical/deployment-automation/), [greater autonomy](https://dora.dev/devops-capabilities/process/team-experimentation/) and so on.

DORA is useful here in showing that such suggestions are supported by the evidence, while the opposite practices (manual regression testing, manual and infrequent deployments) negatively affect a departments performance.

That is to say, the question should not be posed as "what engineers want" versus what "business analysts | QA | etc want". The question is how can we avoid irrational decisions by considering the relevant evidence.

Unfortunately, DORA points at why these conversations can be difficult. There are certain entrenched [cultural factors](https://dora.dev/devops-capabilities/cultural/generative-organizational-culture/) which are difficult to dislodge. While few organizations are pathological, a generative, learning culture is not had by default. It is _achieved_, and only with significant effort.

Rational argument is not enough here. Certain psychological barriers must be surmounted. Evidence must be accompanied by rhetorical persuasion that speaks not only to what is correct, but to the feelings and emotions that may get in the way of objective judgment.

As a result, presenting the evidence and then directly identifying the cognitive biases that surface in response is perhaps not the most successful strategy. Sometimes it is better to just bring donuts.

# Recommended Reading

- Clark, Dominic A. "Human expertise, statistical models, and knowledge-based systems." In Expertise and decision support, pp. 227-249. Boston, MA: Springer US, 1992.
- Dawes, Robyn M., David Faust, and Paul E. Meehl. "Clinical versus actuarial judgment." Science 243, no. 4899 (1989): 1668-1674.
- Grove, William M., and Paul E. Meehl. "Comparative efficiency of informal (subjective, impressionistic) and formal (mechanical, algorithmic) prediction procedures: The clinical–statistical controversy." Psychology, public policy, and law 2, no. 2 (1996): 293.
- Grove, William M., David H. Zald, Boyd S. Lebow, Beth E. Snitz, and Chad Nelson. "Clinical versus mechanical prediction: a meta-analysis." Psychological assessment 12, no. 1 (2000): 19.
- Humble, Jez, and Gene Kim. Accelerate: The science of lean software and devops: Building and scaling high performing technology organizations. IT Revolution, 2018.
