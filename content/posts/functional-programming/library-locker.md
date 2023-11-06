---
categories:
- design
date: "2023-08-20"
description: The library locker antipattern occurs when we try to add functionality to a library function, but do so without separating concerns.
slug: library-locker-antipattern
tags:
- design
- clojure
title: The Library Locker - An Antipattern
draft: false
---

The "Library Locker" is a common anti-pattern for incorporating third party libraries into an application. The application wraps the library with its own function, which mixes concerns and makes the library more difficult to use.

In this article, we will first consider the original problem the library locker is introduced to solve, then an example of the library locker, and then suggest better alternatives.

## The Problem

Suppose we have system of microservices that support astronomical applications. We locate our calls to our other microservices in a `service` directory.

Quickly, we find a certain pattern repeating itself. We need to prepare a request map and parse the response.

```clojure
(ns app.service.night-sky
  "Handles interactions with Night-Sky, a microservice owned by our team"
  (:require [app.auth-tokens :as auth]
            [clj-http.client :as http]
            [clojure.tools.logging :as log]
            [jsonista :as j]))

(defn find-star
  "Find a star in the night sky"
  [{user-id    :user/id
    star-id    :star/id
    root-trace :root-trace/id
    request-id :request/id
    :keys      [originating-system]
    :or        {originating-system "observatory"
                request-id         (random-uuid)}}]
  (log/infof "User %s is requesting star %s. Request id: %s. Trace id %s"
             user-id request-id trace-id)
  (let [response (http/get "http://host.night-sky/api/star"
                           {:oauth        (:night-sky auth/tokens)
                            :query-params {:star-id star-id}
                            :headers      {:user-id user-id
                                           :request-id request-id
                                           :root-trace root-trace
                                           :originating-system originating-system}})
        body (j/read-value (:body response))]
    (log/infof "Received star: %s. Request id: %s. Trace id %s"
                body request-id trace-id)
    body))
```

Most of the body of `find-star` is not about finding a star. It's about logging, tracing, and authentication. What if we wanted to change our logging library? Or improve our log messages?

We need an abstraction. But what should it look like?

## Library Locker Example

We decide to pull out the common code in a single function called `get`. The purpose will be to make get calls to other services we own, while not needing to repeat ourselves in each service call.

```clojure
(ns app.http
  (:require [app.auth-tokens :as auth]
            [clj-http.client :as http]
            [clojure.tools.logging :as log]
            [jsonista :as j]))

(defn get
  "Wrap the clj-http `get` call to unify
  handling of logging, tracing, and authentication.

  Parameters
  ----------
  - `url`: the URL to call.
  - `http-options`: clj-http options"
  [url
   {:keys [query-params] :as http-options}
   {user-id    :user/id
    star-id    :star/id
    root-trace :root-trace/id
    request-id :request/id
    service    :service/name
    :keys      [originating-system]
    :or        {originating-system "observatory"}
    :as options}]
  (log/infof "User %s is requesting star %s. Request id: %s. Trace id %s"
             user-id request-id trace-id)
  (let [response (http/get "http://host.night-sky/api/star"
                           (-> http-options
                               (assoc :oauth (get auth/tokens service))
                               (assoc-in [:headers :user-id] user-id)
                               (assoc-in [:headers :request-id] request-id)
                               (assoc-in [:headers :root-trace] root-trace)
                               (assoc-in [:headers :originating-system] root-trace)))
        body (j/read-value (:body response))]
      (log/infof "Received star: %s. Request id: %s. Trace id %s"
                 body request-id trace-id)
      body))
```

Now, we have a handy function, that looks similar to clj-http's get function, except that it gives us some additional functionality.

This lets us dramatically simplify our `find-star` function:

```clojure
(ns app.service.night-sky
  "Handles interactions with Night-Sky, a microservice owned by our team"
  (:require [app.http :as http]))

(defn find-star
  "What stars are in the night-sky?"
  [{user-id    :user/id
    star-id    :star/id
    root-trace :root-trace/id}]
  (http/get "http://host.night-sky/api/star"
            {:query-params {:star-id star-id}}
            {:user/id       user-id
             :service/name  :night-sky
             :root-trace/id root-trace}))
```

Amazing! Or is it?

Notice that we have _reduced_ the functionality of clj-http's library. `clj-http.client/get` has an async mode:

```clojure
(client/get "http://example.com"
            {:async? true}
            ;; respond callback
            (fn [response] (println "response is:" response))
            ;; raise callback
            (fn [exception] (println "exception message is: " (.getMessage exception))))
```

But the async mode cannot be used. We're locked into synchronous mode.

Moreover, while it was convenient to parse the body of the response, we've reduced the amount of information we return. We only return the body. We couldn't access the headers if we wanted to!

It would be very hard to write `get` using test driven development. And if it's hard to test, it's poorly designed.

Notice that every function call within our `app.http/get` couples us to something:

*Http library*. We're coupled to clj-http. If we were to want to use another library for a special purpose, like aleph, we cannot reuse our `get` function.

*Logging*. While we might easily change the log library, note that we log the entire response. But some responses may be long, or may have information that cannot be logged for security reasons. What if we want to log some of the headers from a certain service? What we shouldn't log a key in the response?

*Parsing*. What if we want to use transit instead of JSON for our responses? What if we want keywordized keys in some cases, but not other cases (where, say, there are keys with spaces in them)?

As the requirements around service calls grow, we will likely see a proliferation of parameters so that `get` can support a broader variety of behaviors. For example, if we want to excise sensitive information from the response body, we may pass a further parameter to `get`. Or, if we want to change the response format, we may pass in a `:response/format` argument.

Extending the behavior of `get` will be error prone. We will be tempted to add defaults, such as a default timeout. This could easily break existing callers. Over time, the function signature will become more and more polluted, and `get` will be more complex to use and understand.

What if we want to add validation on the parameters? Check the response against a contract? Stub out functions for testing?

## Library Locker Definition

We can now offer something of a definition of the library locker antipattern. It occurs when:

- A third party library is wrapped in the application, such that:
- The functionality is reduced or changed, and
- The reduction or change is locked in the scope of the wrapper function, and
- The wrapper function typically extends its behavior by adding parameters

Typically, this will result in a mixing of concerns, code churn on the wrapper function as new capabilities are needed, and the fragility that results from frequent modification.

## Design Principles

Functions should be focused. They should do one thing well. But our `get` function does many things: logging, authentication, parsing, etc.

But our goal is to have a function that we can use in our `services` namespaces to easily call our other services. Is there a better way?

The library locker is a case study in the opposite of the open-closed principle. We should be able to add functionality of our `get` functionality without changing `get`'s source code. Instead, our `get` function serves as a shell that prevents us from extending its functionality.

## Alternatives

Let's consider several alternatives.

### Function Composition

Clojure is a functional language. Consequently, function composition should be top of mind for us in designing flexible software. Without it, Clojure can quickly turn into a scripting language with parentheses.

We notice that we have three types of operations we're doing in our `get` function:

- Manipulation of the request map (e.g., adding headers)
- Handling the response (e.g., parsing the result)
- Doing something with both the request and the response (logging)

We could easily compose functions:

```clojure
(defn trace
  "Trace a request"
  [trace-id req]
  (assoc-in req [:headers :trace-id] trace-id))

(defn identify-user
  "Identify the user to the service we are calling"
  [trace-id req]
  (assoc-in req [:headers :user-id] trace-id))

(defn identify-origin-system
  "Tell the callee service where the request originated"
  [req]
  (assoc-in req [:headers :origin-system] "observatory"))

(defn authenticate-service
  "Authenticate our service with the callee service"
  [service-name req]
  (assoc-in req [:headers :oauth] (get auth/tokens service-name)))

(defn prepare-request
  [{user-id    :user/id
    root-trace :root-trace/id
    request-id :request/id
    :keys      [originating-system]
    :or        {originating-system "observatory"
                request-id         (random-uuid)}}]
  (comp (partial inject-root-trace root-trace)
        inject-origin-system
        inject-oath-token
        (partial inject-user-id user-id))
)
```

There are a few improvements that stand out immediately:

- Composability
- Extensibility
- Declarative abstraction

### Composability

It could easily be the case that requests will have different needs. For example, not all requests may originate from an end user.

By separating behaviors into small, narrowly focused functions, and then composing them together, we can reuse each, and re-compose them in different circumstances.

### Extensibility

Because we are composing functions, we always have the option of further function composition. Particular `service` namespaces, for instance, can compose `prepare-request` to their own unique needs.

Each of our functions has become more declarative. Each of our functions says what it does, not how it does it. Instead of `set-user-id-header`, we `identify-user`. Should we have a different way of doing this in the future, we don't need to change the function name.

This lets us not need to worry about how each function manipulates the request. Our function is more readable by hiding the details. Should anything change about how we identify our user in the future, we probably won't need to change our `prepare-request` function.

Moreover, this composition pattern would work with a number of functions in the same http client library, and by other http client libraries, should we choose to use them.

## Middleware

Function composition is simple, but has some limitations. Note that our functions can access only the request, not the request and response together. If this is a limitation, middleware might be a better pattern.

`clj-http` [supports](https://github.com/dakrone/clj-http#custom-middleware) middleware, but it is simple to roll your own in a library-independent fashion.

## Aspect Oriented Programming

Logging is a cross-cutting concern that can be handled with aspect-oriented approach. It can be handy to instrument functions to capture certain parameters and return values. AOP can be configured differently in different environments as well.

Functions can be made easier to read, because the logging is injected via a library like [Robert Hooke](https://github.com/clojure-land/robert-hooke) or [Hansel](https://github.com/jpmonettas/hansel)

# Objections

There are two common responses to function composition-type patterns:

1. "It's less readable. I can't see what it does! It's more complicated, not less."
2. "Why spend time splitting things up, or establishing library-independent patterns? YAGNI."

These are, I would argue, misunderstandings of what we aim for with readability and YAGNI.

## Readability

If function composition as such is less readable, my suggestion would be that the paradigm of functional programming has not been fully internalized. The trouble is not that function composition as such is less readable, but that a functional language like Clojure is less readable to those who don't understand functional programming.

But the readability problem may go to an even more basic misunderstanding of good software design. Good software design separates "what" from "how". It creates abstractions that allow you to use things by understanding what they are for, without needing to understand the internal mechanisms by which they accomplish it.

Failure to establish this semantic separation means that one has to understand everything to do anything. It leads to situations where, in order to understand something as high level as domain logic, one has to worry about the structure of HTTP calls, database table structures, authorization, etc.

In other words, failure to establish semantic layers is the "easy" path that leads to the big ball of mud. And developers get used to thinking of readable code as an imperative sequence, where all the mechanisms are gathered together.

This is akin to the [blub paradox](https://paulgraham.com/avg.html). When one looks down at lower levels of abstraction, one appreciates the patterns with which one is familiar. But looking up to the levels above what one currently understands, patterns that enforce loose coupling and high cohesion, separation of concerns, etc., one thinks that it's all "just more complex and harder to read."

The solution is not to reduce code quality, but to elevate our own understanding of good software design, both in general software design and in functional programming.

## YAGNI

Why spend the effort to make designs modular and extensible? Must we have a specific use case in mind to justify making code reusable?

If our software can only be used for the use cases we now about now, it will be both brittle and inept. It will be brittle, because we have to alter working code to support new functionality. It will be inept, in contrast to the power of composable, flexible software that is open to cases not initially anticipated.
