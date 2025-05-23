---
title: Several faces of protocols
date: 2020-09-02
author: topolog
layout: post
permalink: /protocol-faces/
image:
  path: images-posts/2020-09-02-protocol-faces/masks-1800.jpg
  thumbnail: images-posts/2020-09-02-protocol-faces/masks-600.jpg
  caption: Photo by Eric Prouzet on Unsplash
tags:
  - iOS
  - swift
  - protocols
categories:
  - Tech Blog
share: true
---

"Protocols is one of the most powerful language features in Swift" - people say. Hard to argue. In Swift you can do a lot of different things with protocols. (Maybe even too many... but we will talk about it later).

Let's try to look at them from a distance and understand what `language features` are masked by this simple word "Protocol" in Swift.


---

**UPD: (Nov 2022)** Now after the number of changes in Swift 5.6 and 5.7 something changed... See [Shift in the Protocol-paradigm](/protocol-paradigm-shift)

---

&nbsp;

_(I don't want to overload you with the code examples here as there are plenty of them out there. But I need to illustrate some ideas with code (and I don't want to just copy-past somebody's examples... even if they are great ;-)).
So let's consider some News Feed app that fetches and displays recent news as our playground.)_

## 1. Protocol as a dynamic interface

First of all, protocol is a good old interface. It's a contract which one object relies on and another object conforms to.

In our News App we have a `NewsProvider`-protocol and `FeedViewModel` which relies on this protocol.

```swift
protocol NewsProvider {
    func fetchNews() -> [News]
}

class FeedViewModel {
    var newsProvider: NewsProvider
    var news = [News]()

    func reloadNews() {
        news = newsProvider.fetchNews()
    }
}
```

We have several screens for different news agencies. For each agency we create a dedicated provider-class that conforms to the `NewsProvider`-protocol. So it can be injected into the view model and can be used as a data source.

```swift
class ReutersNewsProvider: NewsProvider {}
class AssociatedPressNewsProvider: NewsProvider {}
class RIANovostiNewsProvider: NewsProvider {}

class FeedViewModel {
    var newsProvider: NewsProvider
    var news = [News]()

    init(newsProvider: NewsProvider) {
      self.newsProvider = newsProvider
    }
}

let viewModel = FeedViewModel(newsProvider: ReutersNewsProvider())
```

As simple as that. We abstract the interface from the implementation so later we can use various implementations of the `NewsProvider` interface without view model feeling a difference. In lot's of other languages it's called directly "Interface". In Objective-C it's called "Protocol" and Swift inherited the name for this language concept there.

### What do we gain here

This application of protocols in Swift help you to implement [composition](https://en.wikipedia.org/wiki/Composite_pattern) or [delegation](https://en.wikipedia.org/wiki/Delegation_pattern) in your Swift code.

Here we have an example of **dynamic (or runtime) polymorphism**. When `FeedViewModel`-class is being compiled there is a reference to a `NewsProvider`-protocol. When the user enters Reuters page in our app we instantiate a view model and inject `ReutersNewsProvider`. So a specific class appears in the view model only in runtime.

Runtime polymorphism goes together with the dynamic dispatch. In the compile-time, we don't know which exact method of which exact object will be called. It gives some runtime flexibility but blocks the compiler to do most of the compile-time optimizations.

With dynamic polymorphism, we can also create a heterogeneous collection. For example, our `FeedViewModel` can hold an array of different news providers and fetch news from all of them if we decide to design an aggregation screen in our app.

```swift
class FeedViewModel {

    var newsProviders = [NewsProvider]()
    var news = [News]()

    func addNewsProvider(provider: NewsProvider) {
        newsProviders.append(provider)
    }

    func reloadNews() {
        for provider in newsProviders {
            news.append(contentsOf: provider.fetchNews())
        }
    }
}
```

## 2a. Protocol as a compile-time constraint

Other than traditional dynamic polymorphism, protocols can be also used for static polymorphism. In this case, a protocol is resolved to a specific data type not in runtime, but in compile-time.

Let's rewrite our `addNewsProvider`-function this way:

```swift
class FeedViewModel {

    var newsProviders = [NewsProvider]()

    func addNewsProvider<T: NewsProvider>(provider: T)  {
        newsProviders.append(provider)
    }
}
```

Functional-wise nothing changed but now we use generics. Instead of saying that we pass "something conforming to NewsProvider" we say that "we pass the exact type T which conforms to NewsProvider". Now we use Swift generics system. For this piece of code where `T` is a parameter, it's still dynamic polymorphism: the real type appears here only in runtime. You can still keep this `newsProviders: [NewsProvider]` for example.

But in the client code now the compiler has to resolve it to a specific type. So you cannot do something like that:

```swift
let array: [NewsProvider] = [ReutersNewsProvider(), AssociatedPressNewsProvider()]

viewModel.addNewsProvider(provider: arr.randomElement()!)
```
It's impossible to know in the compile-time what specific provider will be passed to a view model, so the compile prohibits it.

We can still use 100% runtime polymorphism for this protocol when the flow doesn't touch generics:

```swift
let array: [NewsProvider] = [ReutersNewsProvider(), AssociatedPressNewsProvider()]
var provider: NewsProvider?
provider = arr.randomElement()!
```
Here the compiler doesn't mind not knowing the actual type of `var provider`.

## 2b. Restricted protocol as a compile-time constraint

As soon as we add an associated type to the protocol or `Self`-constraint your protocol loses all the dynamic polymorphic capabilities.

```swift
protocol NewsProvider {
    associatedtype NewsModel

    func fetchNews() -> [NewsModel]
}
```
Now your protocol cannot be used other than as a generic constraint. All that cannot be checked in the compile-time is prohibited. Now the contract gets so complicated that it cannot be resolved other than figuring out the exact type which you want to where this protocol is expected. No more dynamic polymorphism. No more `var provider: NewsProvider?` or `newsProviders: [NewsProvider]`. Every time you try to do something like that you receive the notorious error:

> Protocol _'YourProtocol'_ can only be used as a generic constraint because it has Self or associated type requirements

You can either use some constraint type `T<NewsProvider>` (when possible) which will be resolved to a specific type by the compiler, so you accept these static rules). Or you can use [type erasure](https://www.google.com/search?q=type+erasure+in+Swift) and insist on some runtime logic in resolving the types.

If you are not sure what "Self-constraint" is take a look at `Equatable`:

```swift
public protocol Equatable {
    static func == (lhs: Self, rhs: Self) -> Bool
}
```

In every specific case of conforming `Self` will be replaced to a specific data type (as you cannot compare two different ones even if they both conform to Equatable). So it's not so wrong to say that `Self` behaves here the same way as `associatedtype` does.

BTW, some of these runtime-restrictions are related to something called "existentials". This concept is used in other programming languages and was mentioned in the [Generics Manifesto](https://github.com/apple/Swift/blob/master/docs/GenericsManifesto.md#existentials) and there were [at least one discussion](https://forums.Swift.org/t/status-of-generalized-existentials/13982) and [a proposal](https://github.com/austinzheng/Swift-evolution/blob/az-existentials/proposals/XXXX-enhanced-existentials.md) how it can be solved in Swift. But seems like there is no further work in this direction.

### What do we gain here

Associated types and Self-constraint (which kinda associated type as well ;-)) are something that allows us to do all this protocol-oriented (or generic-oriented) programming. It allows us to write flexible code which can abstract huge chunks of functionality from the specific types, so they can be reused.

With such a powerful concept you can abstract, for instance, your [data store logic](https://faical.dev/articles/modern-Swift-data-access-layers.html), your [table view logic](https://github.com/maxsokolov/TableKit), or every here and there for [abstracting smaller pieces of logic](https://www.Swiftbysundell.com/articles/specializing-protocols-in-Swift/).

The price of this abstraction is quite high. It's not only the limitations we just mentioned but also some additional cognitive load (hope to tackle it more in detail in one of the next posts).

## 3. Protocol as a code generation directive

Sometimes protocol can be used additionally to tell the compiler to generate some code. Usually, that's some boilerplate code for the type conforming to this protocol.

For instance, if you male your model to conform to `Codable` the compiler will generate the implementation of Encodable/Decodable functions for you. The same happens with `Equatable`. Implementation can be generated if all the nested data types conform to the protocol. If you need to add some logic you are free to reimplement the protocol-methods yourself, but in the majority of cases it's not needed.

If you are interested in how the compiler does it check out [this (as always) brilliant article at NSHipster](https://nshipster.com/Swift-gyb/)

### What do we gain here

Protocols in these cases still play their role as interfaces. Some mechanisms from a standard library need your custom data types to conform to these protocols. But the code generation is a nice addition which makes you free from implementing the same code again and again. Of course, it's applicable only to specific use cases when implementation follows some pattern so can be easily automated. (Think about your own protocols, maybe it could make sense to use some code generation for some of them as well ;-) )

## To be continued...

I suppose you noticed that we haven't said a word about `extensions` here, although a lot of folks think they a necessary part of "Protocol-Oriented development" (I'm not among them). Here is the [next post dedicated to the protocol extensions](https://dmtopolog.com/protocol-extensions/).
