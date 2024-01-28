---
categories:
- Architecture
- Clojure
date: "2024-01-27"
slug: clojure-uis-hypermedia-and-rpc-2
tags:
- engineering
- hypermedia
- rest
title: The False Dichotomy
draft: false
---

*This is the second post in the series "Have Clojure UIs Taken the Wrong Path?". The first post is [here](/2023/11/clojure-uis-hypermedia-and-rpc-1/).*

How do we choose between building an application using a hypermedia approach versus the client side SPA? This post won't answer that question. But it will say now *not* to make that choice.

It is pretty common to hear that, while old fashioned "Multi Page Apps" (MPAs) are OK for some simple use cases, Single Page Applications are better for rich client interactivity.

<img src="/img/the_spectrum.svg" alt="mpa-svg-spectrum" style="max-width: 999px;">

On the one hand we have multi page applications (MPAs). These are the "old fashioned" web applications that serve HTML. On the other hand, there are client side single page applications (SPAs). These use React, Angular, Vue, Svelte, or some similar to replace core browser functionality such as navigation and state management. Depending on how much interactivity you want your app to have, you select from along the spectrum, and you make the tradeoffs.

Any time you are told that "it's a matter of tradeoffs", alarm bells should go off. It's a good indicator that the approach is fundamentally misguided. Good software design increases your options. Rather than reducing your options, well designed systems increase your optionality. Posing the choice between something that is faster to build but less capable (MPAs) or more capable but complex (SPAs) is just such an example.

Moreover, as Clojurians, shouldn't we be suspicious of an approach that argues that sophistication requires *abandoning simplicity and data-orientation*? The SPA-RPC approach, as I pointed out in [part 1](/2023/11/clojure-uis-hypermedia-and-rpc-1/), couples UI and domain concerns. It does so by abandoning the commitment to data orientation. Instead of shipping information and controls as data, we now need megabytes of minified JavaScript. Abandoning data orientation and choosing to complect UI and domain concerns are intrinsic to the RPC architecture. 

Moreover, the constraints that cause the tradeoffs in the first are often entirely accidental to the problem space. Instead, those tradeoffs come from approaches that *reduce our options*. What if, instead, we could take an entirely different approave that offers simple, extendable, composable parts? What if we could opt in to complexity where we need it? Could we keep the complexity local so that it does not spread to the rest of the application?

We'll expand on these options below. But first, let's consider another misleading aspect of the spectrum: that client-side SPAs are needed for maximal client interactivity. In fact, it is often the need for extensive, app-like interactivity *rules out* client side SPAs. An example will be useful to illustrate this point.

## Client Interactivity

I once built a browser-based live sports telestration application. Telestration involves adding animations onto a screen. Here's a demonstration of some of its features.

<video src="/video/telestration_clip.mp4" controls></video>

This involves some pretty complex client functionality. The animations must be aware of the players on the field. The spinning circles should be under the players' feet flat on the field. The arrows to be aware of the angle of the field, even when the camera pans. Users can voice over the video. When they export the video, the audio of their voice is combined with the video and the animations. While users are editing the videos, they want to be able to preview the animations and undo mistakes. The application is used both in live settings (team practices, Zoom calls), and to export to videos that can be shared. Teams don't always have great wifi, so the client needs to be able to recover from failed network connections.

This is not a case for React. React is far too slow to sustain a good framerate on a high definition video. The state includes every individual pixel on the original video, plus every pixel on the animations, plus the audio from the microphone. The logic involves applying algorithms on each frame to determine where players are, and what the lay of the field is relative to the camera. Some of this is in JavaScript, some is in WebGL.

But frameworks that don't have React's performance overhead aren't fits either. React is not a fit *because of* the complexity of the client state. The state management and logic is tied closely to the video frames and WebGL.

What lesson can we draw? Maximal client side interactivity requirements are likely to disqualify JavaScript SPA frameworks.

## Line of Business Applications

Let’s go to the other end of the spectrum and look at line of business applications. Suppose we don't need any complex client interactivity. Is it really a good idea to develop these using just HTML? Here too the spectrum spectrum of tradeoffs fails to guide us to a good decision.

Users don’t want full page reloads every time they submit a form. They like live updates that show up without needing to refresh the page, inline editing, dynamic data tables, and infinite scroll. Even in cases where developers are told that the UI doesn’t need to be pretty, it is inevitably the case that quality is judged largely on appearance.

The old fashionied way is not just a poor experience for end users. Developers want to go back to the days sprinkling in JQuery. Features like hot reload are nice!

Wouldn’t it be nice where we had an incremental approach? One that begins easy, with the affordances users and developers have come to expect? On that allows us to add sophistication only where we need it? Instead of architectures that constrain our options, why not seek architectures that maximize our options? Here we return to the notion of extended hypermedia.

## Hypermedia is not HTML

HTML is a successful hypermedia. But it is important not to confuse HTML with hypermedia.

Let’s reconsider the definition of hypermedia offered by Roy Fielding:

> When I say hypertext, I mean the simultaneous presentation of information and controls such that the information becomes the affordance through which the user (or automaton) obtains choices and selects actions. Hypermedia is just an expansion on what text means to include temporal anchors within a media stream; most researchers have dropped the distinction.

Interactivity is built in to the definition of hypermedia. That’s what makes it *hyper*.

For the rest of this article, we’ll use hypermedia in the following strict sense. Something is hypermedia if and only if:

1. It is data interchanged between a client and a server

2. The data includes all the information the user needs to see and the controls for the user to take actions

3. The client has no implementation-time knowledge of domain

### Data Oriented

It is data oriented. By contrast, consider a Re-frame application. In order for the user to see or do anything, the browser receives two things from the server: data (usually in JSON or EDN form), and code (minified JavaScript). The data does not determine what the app does. On its own, it does nothing: it needs a (large) client to render anything to the user.

A hypermedia approach, on the other hand, sends the application as data to a generic client. It encodes both the information the user needs to see, but also – crucially – the controls: what the user can do.

### Simple

Out of this data orientation arises simplicity. The hypermedia client is completely decoupled from specific application domain. A hypermedia client knows how to transform hypermedia data into a UI for the user, without knowing anything at all about the application domain.

The problem is that the range of hypermedia supported “out of the box” by the browser is very limited. It provides a poor experience to users, and isn’t much better for developers.

## Extended Hypermedia

In the extended hypermedia model, the browser is extended as a hypermedia client. There is nothing intrinsic about hypermedia – or HTML – that requires full page reloads or that prevents infinite scrolling. The lack of support is just an implementation detail of the browser.

But we have various means to extend the browser as a hypermedia client. One of those is JavaScript. HTMX is written in JavaScript, yet it extends the hypermedia model. (Examples)

Consider an example where we have a task status, and we want it to update without refreshing the page. In HTMX, we can do something like this:

```html
<span hx-get="/task/TSK001/status" hx-trigger="every 10s">
  Assigned
</span>
```

The client will poll the backend every 10 seconds to see if the task status has changed.

Let’s review our criteria:

✅ Data shipped between a client and server.

✅ This is data oriented. Everything the user needs is shipped in band.

✅ No implementation time knowledge is needed on the client side. 

But didn’t we add HTMX? Isn’t that a client-side change?

This gets at the key point: the browser has been *extended* with HTMX. Instead of treating the browser as a runtime for a custom application, we can treat the browser as an extensible hypermedia client. The browser + HTMX is a hypermedia client. It has no implementation-time knowledge of our use case.

JavaScript is not, therefore, opposed to a hypermedia architecture. It’s a powerful way to extend the browser as a hypermedia client!

But JavaScript is not the only mechanism for extension. What if we need custom DOM elements? Web components let us create new, custom DOM elements. What if we need offline functionality? Obviously JavaScript can be used. But recall that one constraint of REST is that works with layered systems. Service workers could return HTML when the network is down to indicate that a form needs to be re-submitted.

Then there is the Phoenix Liveview model. Liveview is an SPA, but one where the state is controlled on the server side. As the state changes, efficient HTML diffs are sent over a websocket, and Morphdom is used to quickly modify the parts of the DOM that need to change.

Clojure has some libraries that take the Liveview approach. [Ripley](https://github.com/tatut/ripley) is one such library. 

```clojure
(def counter (atom 0))

(defn counter-app [counter]
  (h/html
    [:div
      "Counter value: " [::h/live {:source (atom-source counter)
                                   :component #(h/html [:span %])}]
      [:button {:on-click #(swap! counter inc)} "increment"]
      [:button {:on-click #(swap! counter dec)} "decrement"]]))
```

Notice that the `counter` atom, and the swap operations exists *on the server side*. 

## The One True Way

The point here is not to say that HTMX, web components, service workers, or server-side SPAs are "the one true way". The way I am advocating is not tied to a particular technology, but a particular way of evaluating technologies.

- Choose the options that increase your options
- Use simplicity, loose coupling, and high cohesion to avoid complexity
- Prefer data-orientation

Is hypermedia the only way to do this? No. [Electric](https://github.com/hyperfiddle/electric) is one example of a promising (though new) alternative.

But using the browser as an extended hypermedia client has a lot going for it. It's easy to get started with, and you can extend it in almost any way you need. It is very well understood. Great tooling exists. Documentation abounds.
