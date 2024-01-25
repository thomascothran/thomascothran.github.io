---
categories:
- Clojure
date: "2023-11-05"
description: "Andy Hunt and Dave Thomas tell us: 'it’s critical that you write code that is readable and easy to reason about.' This seems uncontroversial; it is the rare point on which software engineers typically agree. Or do they?"
slug: readability-and-functional-programming
tags:
- abstraction
- clojure
title: The Wrong Kind of Readability
draft: false
---

In the Pragmatic Programmer, Andy Hunt and Dave Thomas tell us: “it’s critical that you write code that is readable and easy to reason about.” This seems uncontroversial; it is the rare point on which software engineers typically agree. Or do they?

In fact, developers disagree about what “readability” means. "Readability" can be given two contrary meanings that we will call imperative readability (or readability-i) and declarative-readability (or readability-d).

## Imperative Readability

This form of readability focuses on how the code operates. It emphasizes a clear presentation of the implementation details, such as which libraries are used, how data sources are accessed, and how functions perform their tasks. 

Code that is "readable-i" is usually more literal and concrete, often resembling procedural code where the sequence of operations and the flow of data are straightforward to follow.

Imperative readability attempts to answer the questions that bear on how code works. 

- How does the function execute its logic?
- What are the specific steps taken by the code?
- Which operations are performed in what sequence?
- How are state changes managed and tracked?

These questions are answered by exposing implementation details in a clear, organized way.

Architectural layers and abstraction impedes imperative readability, since both hide the concrete implementation details.

## Declarative Readability

On the other hand, declarative readability (or readability-d) uses abstractions and architectural layers to manage complexity. It seeks to answer very different types of questions:

- What is the intent of this function or module?
- What business rule or domain concept is this code encapsulating?
- What is the end goal or output of this code, in terms of the domain?
- What are the high-level policies or constraints that this code is adhering to?

## Why does it matter?

If a Clojure code base as a whole is readable in the imperative sense:

- Our Clojure code will look a lot like Java 7, albeit with parenthesis and immutable data structures. We won't use functional programming techniques like function composition and higher order functions. Nor will we make use of polymorphism. Our namespaces are basically classes and our functions static methods. They will be combined in imperative fashion.
- The lack of architectural layers leads to Spaghetti code. For example, our business rules and our use cases will tend to be tied to our data sources. Our data layer will be at the center of the application, since we lack the facilities to abstract and invert the dependencies.
- Our code base will not be expressive of the domain. Code that lacks declarative readability requires developers to understand the domain independently of the code base. Code that is readable-d teaches you the domain.

If you add a new member to the team, can you point them to a section of the code base that clearly expresses the domain? Or do they need to learn the business domain through osmosis?

Are there specific namespaces in your code base that clearly answer the question: "in the given business process, who can do what, under what circumstances, and for what reason?" Or do you have to sift through the UI controllers that enable and disable buttons and form submissions?

## An Example

To spell out the difference, we will use an example based approach. Our hypothetical application is a financial services application that, among other things, handles loans and loan applications. 

It uses a microservices architecture. Our first example will concern calls between services.

```clojure
(ns finco.loans
  (:require [org.httpkit.client :as http]
            [next.jdbc.sql :as sql]
            [taoensso.timbre :as timbre]
            [jsonista.core :as j]
            [jackdaw.client :as kafka-client])
  (:import [java.time.temporal ChronoUnit])

(defn approve-loan
  [{:keys [auth-token] :as config}
   {:keys [jdbc-conn kafka-producer] :as connections}
   {:keys [state first-name last-name date-of-birth debt income] :as applicant
   {:keys [trace-id username user-id] :as ring-request}]
  (let [state-minimum-age 
        (-> (sql/find-by-keys jdbc-conn 
                              :state_minimum_loan_ages
                              {:state state})
            first 
            :state_minimum_loan_ages/minimum_age)

        years-old (.between ChronoUnit/YEARS 
                            date-of-birth 
                            (LocalDate/now))

        old-enough? (<= state-minimum-age years-old)
        
        _ (timbre/info "Requesting credit score for " first-name " " last-name ". " trace-id)

        credit-score
        (-> @(http/get "https://fincoloans.tech/api-gateway/credit-scores"
                        {:query-params {:first-name first-name 
                                        :last-name last-name
                                        :dob date-of-birth}
                         :headers {"Authentication" (str "Bearer " auth-token)
                                   "Trace-ID" trace-id
                                   "Username" username
                                   "UserID" user-id}})
            :body 
            j/read-value)

        sufficient-credit? (< 650 credit-score)]

     (if (and old-enough? sufficient-credit?)

       (do (timbre/info "Loan approved " applicant " - " trace-id)
           (sql/insert! jdbc-conn :loan_approvals (merge applicant loan))
           (kafka-client/produce! kafka-producer 
                                  "loan-approvals" 
                                  (j/write-value-as-string (merge applicant loan))})
           :approved)

       (do (timbre/info "Loan Denied " applicant " - " trace-id) 
           (kafka-client/produce kafka-producer 
                                 "loan-denials"
                                 (j/write-value-as-string (merge applicant loan))
           :denied)))))
```
We are not handling errors, and the actual business logic of a use case like this is more complex, but you get the idea.

This function is readable-i, but not readable-d.

### Imperative Readability

It is readable-i because it clearly expresses the details of how it works. We can learn quite a bit by reading the function:

- What the data sources are. We use http to request the credit score from one of our other microservices, rather than fetching it from a database or using an external service. We use a database table for the minimum age for a state.
- How authentication is done handled between services. We use a bearer token and add the username and user id to track just who is requesting the credit score.
- How tracing is implemented. We are using timbre and adding the trace in the logs.
- What libraries we use: http-kit, timbre, and jsonista.

Readability-i is an answer to the question "how does this work?". All the implementation details are "out front". One reads them and infers what the program is for.

Now, clearly our `approve-loan` function is poorly written. Functions should "do one thing", but our function does a number of things. It calls various services, handles cross-cutting concerns like logging and tracing, implements authentication, and updates the database and Kafka (in a non-transactional way).

What's important to note is the reason `approve-loan` is poorly written: precisely because it is readable-i. This means that if we expect functions in our code base, especially the high level use cases, to clearly express how they work, then we are the problem.

To repeat: the insistence on code being readable (in the sense of expressing how it works, preferring concrete implementations to abstractions) produces bad software. Imperative readability is easy, not simple.

#### Abstraction

By contrast, declarative readability is effective because of its abstractions. Abstraction is not a programming-specific idea. It is best illustrated by scientific discovery. In scientific discovery, the inessentials are set aside, the essential is grasped, and the essential is articulated in the form of a theoretical model.

For example, Newton noticed that apples always fall perpendicular to the ground. His insight was into the idea that all things with mass might obey the same law of attraction: whether they be apples, planets, or tides. In formulating his law of gravitation, Newton was able to set aside the inessential (whether a thing is red or yellow, small or large, moving or still); what matters is simply a thing's mass.

But the real moment of abstraction is not subtractive, it is creative. It is the production of a mental construct that brings order. In Newton's case, this is the law of universal gravitation: the force of gravity is proportional to the product of the mass of two objects divided by distance between their centers.

The scientific example is not merely illustrative. The ability to abstract is what makes human beings intelligent. Recognizing patterns and repeating them to "get things working" are capacities we share with animals. But the construction of meaning -- of theorems, explanations, and explanatory models -- belongs to higher, human intelligence.

#### Abstraction Applied

In the context of programming, the subtractive is important but secondary to the constructive moment. Nevertheless, the question "what is inessential" is the best place to start.

What do we not need to know about `approve-loan` to accomplish our purpose?

- How requests are authorized. This can be shuttled off to a function.
- How logging is implemented. This is a cross-cutting concern.
- Whether we use a database, a microservice we own, or an external microservice as a data source.

Let's take a ports and adapters approach.

```clojure
(ns finco.loans)

(defmulti get-credit-score
  (fn [data-source {:keys [state first-name last-name date-of-birth] :as applicant}]
    (class data-source)))

(defmulti state->minimum-age
  (fn [data-source state]
    (class data-source)))

(defmulti create-approval!
   (fn [data-source applicant loan]
     (class data-source)))

(defn approve-loan2
  [data-sources
   {:keys [state first-name last-name date-of-birth debt income] :as applicant
   {:keys [amount] :as loan}
   {:keys [on-success! on-denial!]}]
  (let [years-old (.between ChronoUnit/YEARS date-of-birth (LocalDate/now))

        old-enough? (<= (state->minimum-age (:state-laws data-sources) state)
                        years-old)
        
        sufficient-credit? (< 650 (get-credit-score (:credit-scores data-sources) applicant))]

     (if (and old-enough? sufficient-credit?)

       (do (create-approval! (:applications data-sources) applicant loan)
           (when on-success! (on-success! applicant loan))
           :approved)

       (do (when on-denial! (on-denial! applicant loan))
           :denied))))
```

Notice how simple this is compared to our previous implementation. Previously we were coupled to everything from a particular http library to the format of the responses from our services. We had to worry a great deal about how things work. Now we can focus on the application logic. 

Our application logic is stable. We could swap out the source for where we get our credit scores, or we could split out further microservices. If we decide to add a cache, we could use `on-success!` to populate it.

In removing the inessentials, the expression of the essentials becomes much more clear. A developer who knows nothing about loans could get a good idea of what the business logic is just by glancing at this use case.

### Simple / Easy

A developer who cannot think abstractly may have trouble with `approve-loan2`. "Why can't I jump to the source?" "What is this really doing?" But these are the wrong kinds of questions to be asking at the level of the business rules or the application logic. 

But in practice, defenses of readability-i usually come in in the practice of *writing* code. It is certainly the case that what is easier to write in the moment takes less work than forming the right abstractions, thinking through semantic layers, and so on.

But this is hard not because it is more typing, but because thinking through things intelligently, formulating ideas clearly and rigorously is hard work. It is easier to follow a pattern, easier to "get things working", and delegate the understanding of a business to a product owner or business analyst.

Here, the problem is not with abstractions. It is with ourselves, and our ability and propensity to use our intelligence when we program. The real challenge is to change more than our code base -- it is to improve our habits and our mindset. 

## Some Quick Pointers

To start to shift from the imperative to the declarative mindset, here are a few places to start.

### Embrace functional programming

Functional programming is not writing functions that take and return data. Java 7 is perfectly suitable for that (substituting classes for namespaces and static methods for functions).

In functional programming, functions are first class. Functions can take and return other functions. They can be stored in data structures. 

To understand why this is important, ask yourself: how would you implement `map` if functions could not take other functions as parameters? How would you implement `juxt` without being able to return a function from another function?

Do `comp` and `partial` feel foreign? Does passing in a function to facilitate testing feel strange? 

If higher order functions aren't in your toolbox, it's probably a good idea to work on functional programming concepts.

### Polymorphism for Data Sources

Side effects can be isolated from pure functions, but while business rules can be implemented without reference to IO, the same cannot be said of all domain logic. Application logic, such as our `approve-loan` example above, needs to do IO.

However, the indepenence of the domain logic can be maintained using dependency inversion. We used a ports and adapters style approach, but there are many variations (clean architecture, onion architecture, etc).

### Brush up on software architecture

Serious study of software architecture is important. Many of these texts tend to use object oriented approaches, but the core principles are applicable to Clojure as well.

Classics like "Domain Driven Design" by Eric Evans bear re-reading. Plenty of newer texts, like Sam Newman's work on microservices, are worth the time as well.

A good grasp of architecture is more important in Clojure given its dynamic nature than in statically typed languages. It's easy to respond to the newfoud freedom from a type system by churning out spaghetti.
