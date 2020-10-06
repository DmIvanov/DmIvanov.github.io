---
title: Do protocols break Single Responsibility Principle?
author: topolog
layout: post
permalink: /do-protocols-break-srp/
image:
  path: images-posts/2020-10-06-do-protocols-break-srp/bubble-1920.jpg
  thumbnail: images-posts/2020-10-06-do-protocols-break-srp/bubble-600.jpg
  caption: Photo by Diana Orey on Unsplash
tags:
  - iOS
  - swift
  - protocols
  - patterns
categories:
  - Tech Blog
share: true
excerpt: In previous posts we've mentioned all the different use cases for this language feature. Now let's consider some hidden complications which we get together with all the power.
---

This post continues the ideas mentioned in the previous ones from this series:
- [Protocol extensions](https://dmtopolog.com/protocol-extensions/)
- [Several faces of protocols](https://dmtopolog.com/protocol-faces/)
- [Pitfalls of protocol extensions](https://dmtopolog.com/pitfalls-of-protocol-extensions/)

The power of the protocols in Swift is their main weakness. There are too many use cases for protocols, they try to embrace too many different language features, **too many responsibilities**.

I'm quite far from designing programming languages, but as I understand it there is a kind of a dichotomy regarding every language feature.

On the one hand, every feature has to be designed as generic as possible. So with the least amount of features you can cover all the use cases for the language users. The language is usually considered too verbose if it has too many language concepts. Also the more concepts it has the harder it is to learn it. Everybody wants to attract more people to use the language (it's not as simple of course, but there is much truth in this statement).

On the other hand, with too generic features come some specific problems. One of them is breaking the Single Responsibility Principle (SRP).

Let's remember why we all appreciate SRP in the first place. Why do we apply it to our functions and data structures:
- It makes code simpler and easier to comprehend
- It minimizes the number of side effects

SRP is quite a universal design idea so in language design, it means exactly the same.

_Here we omit all that is related to the internal implementation of the feature and just focus on the external side of the deal: the language as a tool._

Stuffing several concepts in one language feature breaks the SRP. The feature gets more difficult to understand. It is like a word with several meanings: you need to grasp a fair amount of context to know exactly what is the meaning in this very case. It increases complexity and requires more cognitive resources.

By breaking SRP we increase the number of side effects. And in the case of language concepts, it means that one use case for the feature language can interfere with another.

That's basically what happened with **inheritance**. We use one term (and one language concept) for two different things: _interface inheritance_ and _implementation inheritance_. When not separated in the language, they cause some troubles (if you are not familiar with the problem you can [find a lot of discussions in Google](https://www.google.com/search?q=inheritance+is+evil)). Developers mix one with another in the same data structure because it's so easy to do that. You might stick to the interface inheritance in your class and refuse to use the implementation one to keep your code clean. But one day you decide to override one of the methods in a subclass (not a big deal). Then you put instances of a superclass and subclasses into the same array (we like generic code, right?). But next day you already find yourself down-casting a superclass

```swift
for animal in animals {
    if let dog = animal as? Dog {
        // you can pat it
    }
}
```
That's not where you want to be.

Back to protocols.

We mentioned all the different responsibilities of protocols and protocol extensions in [Several faces of protocols](https://dmtopolog.com/protocol-faces/) and [Protocol extensions](https://dmtopolog.com/protocol-extensions/).

It's easy to mix several roles in one single protocol. Let's say, you start with a simple dynamic interface. Then you find some common implementation patterns in types conforming to the protocol so you add an extension and put all this shared code into it (why to duplicate the implementation in different places). Then you put some additional functions to an extension that is not in the protocol at all (some free functionality, nice!). It's already quite a complicated piece of logic, but you may subclass a class which adopts this protocol, or inherit the protocol in another protocol, or even make another extension to this child protocol where you override some functions...

In these cases, you eventually have this Frankenstein monster with several responsibilities, quite a complex and even confusing logic, some hidden functionality. But the worst thing is your inability to change a small thing because it has side effects and breaks something else (related to a different responsibility of the object).

> Error: Protocol ‘YourProtocol’ can only be used as a generic constraint because it has Self or associated type requirements

We already mentioned that this multi-concept situation is not good for readability either. Sometimes when looking at the specific protocol it's hard to tell what is its purpose. Would it really be more cognitive load to handle several different language features, then to handle the one, but with several flavours and separate use cases (which sometimes don't work along so well)?

Wouldn't it be better to have different language tools for different use cases? Is it possible to split already released (and highly appreciated) language feature into several independent ones? In the case of protocols, I don't think it's possible at this stage of language development.

I want to briefly remind you that such a "split" already happened in Swift history. Some of you might remember that before Swift 2.2 there were two different language features named 'typealias'. But then it was decided to separate them. That's how we got `associatedtype`. (You can brush up the memories [here](https://github.com/apple/swift-evolution/blob/master/proposals/0011-replace-typealias-associated.md))

No doubt that protocols are much more massive in terms of compiler logic and usage in our code. It's also clear that what was possible in Swift 2 times might not feasible now. I'm just saying that sometimes this situation when several concepts appeared to be encapsulated in one feature it might be realized and fixed at some point.

The conclusion here is simple: let's try to be more aware of what we use under the hood when we write our code. It's great to know our tools and language possibilities, but even better to be conscious about the language concepts underneath.

And remember about SPR. It's like with drinks: better not to mix ;-) Cheers! 🍻
