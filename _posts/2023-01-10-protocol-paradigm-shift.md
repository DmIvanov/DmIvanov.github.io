---
title: Shift in the protocol paradigm
author: topolog
layout: post
permalink: /protocol-paradigm-shift/
image:
  path: images-posts/2023-01-10-protocol-paradigm-shift/butterfly-1920.jpg
  thumbnail: images-posts/2023-01-10-protocol-paradigm-shift/butterfly-600.jpg
  caption: Photo by Suzanne D. Williams on Unsplash
tags:
  - iOS
  - swift
  - protocols
  - obj-c
categories:
  - Tech Blog
share: true
---

In this article we will be talking about the recent changes related to protocols: opaque, existential and generic types; `some` and `any`; runtime and compile-time polymorphism.

I won't describe the details of those features, but will share my thoughts on how they change the perception of protocols, and why we can finally say that protocols in Swift are completely different from what they are in Objective-C.

Those changes originated from the discussion initiated by Joe Groff at the forum (back in April 2019 ["Improving the UI of generics"](https://forums.swift.org/t/improving-the-ui-of-generics/22814)) and have been gradually released in 5.x Swift versions. *(The initial post was itself a result of previous discussions among the members of the Core team and can be traced back to 2016, the time in between Swift 2 and Swift 3 when [Generics Manifesto](https://github.com/apple/swift/blob/main/docs/GenericsManifesto.md) was created)*.


### The dualism

From the origins of Swift, *protocol* (as a language feature) was always in between two completely different worlds: compile-time constraints and runtime flexibility.

#### Runtime polymorphism

From Objective-C protocols inherited their dynamic essence. In this meaning "protocol" is what called "interface" in most of other languages. It's a capability to define some contract (variables, functions) which then will be implemented by specific data types. So the compiler doesn't know about a specific type, and exact implementation of the contract is being "attached" only in runtime using *dynamic method dispatch*. It's more flexible, but less specific and impossible to optimise by the compiler.

Here is an example of runtime polymorphism:

```swift
protocol Animal {}
class Cat: Animal {}
class Dog: Animal {}

var someAnimal: Animal = Cat()
someAnimal = Dog() // we can reassign a value of another type to this variable
var animals: [Animal] = [Cat(), Dog()]
```
`Animal` is a specific type both for the variable `someAnimal` and for the element of the `animals` collection. Compiler has no idea about the exact types regarding those vars. The exact type doesn't matter here at all. So protocol behaves like a type itself. This type is called **existential type**. As you can see variable of existential type can hold values of different concrete types as far as they conform to the protocol. Collection of existentials can be heterogeneous, meaning it can also hold values of different concrete types.

```swift
func pat(animal: Animal) {
    // `animal` has a value of existential type `Animal`
}
```

#### Compile-time polymorphism

At the same time in Swift world protocol always played a big role in *generics*. Generics is a system that allows us to specify a set of requirements/constraints that can be later translated by a compiler into specific types. As the types are clear in compile time, the implementations are "connected" to the calls at that stage as well, so *static method dispatch* is being used. Hence the compiler can help you a lot here during development as well as optimising the code while compiling it.

```swift
func pat<T: Animal>(animal: T) {
    // `animal` is a value of some specific type, conforming to `Animal`
}

pat(Dog())
pat(Cat())
```

**Generic type** `T` is being resolved into a specific type for each function call. So we know the exact type (`Cat` or `Dog` in this case) inside the `pat()` function. In this simple example it doesn't bring us much value, but in a more complex construction it may have a big difference.


#### Alternative terms

On the Swift forum you may also encounter such terms as **type-level** and **value-level abstraction**. They describe the same dualism from the perspective of the compiler. 

Generic types (that represent compile-time polymorphism) provide abstraction on a type level so the compiler resolves the abstract types and then can already operate specific types, completely preserving their details (functions, properties). So we have **type-level abstraction** here.

In case of existentials and runtime polymorphism the exact types covered by protocols are not known for the compiler, so they are replaced by existential types (one protocol - one existential type). So the compiler have only ideas about the existential type and the values that will be assigned to it in runtime. Hence **value-level abstraction**.

In [this forum post](https://forums.swift.org/t/improving-the-ui-of-generics/22814#type-level-and-value-level-abstraction-1) you can read more about it.


### The paradox

This may not sound like an important thing, but there was a lot of confusion and frustration among the developers regarding the dualism of protocols, up until recently (before the discussed changes were released). Sometimes you were not allowed to use the same protocols in different use cases mixing runtime and compile-time capabilities. In some situation it could also cause some errors and unexpected behaviours (both in runtime and in compilation time). *(In our previous articles ["Several faces of protocols"](/protocol-faces/) and ["Do protocols break Single Responsibility Principle?"](/do-protocols-break-srp/); we dived into the subject in more details)*

There were always different opinions inside the Swift Core Team and most active part of the community regarding Protocols. Some people admitted that there was this feature dualism between the runtime and compile-time capabilities. For instance Dave Abrahams [in one of the discussions on the forum](https://forums.swift.org/t/lifting-the-self-or-associated-type-constraint-on-existentials/18025/42) said:
> I have always thought a big part of our problem is that protocols that are meant to be used for type erasure are fundamentally different from those meant to be used as constraints, yet we declare them the same way.

*(The author of this post also belonged to this camp, as you could guess from the previous articles. We even argued that it could have been better to have two separate language features instead of one).*

But the majority considered all the capabilities of protocols as one big feature, that temporary had some gaps and contradictions.

Since the first versions, protocols in Swift played more important role as generic types other than existential ones. Runtime polymorphism is less safe, predictable and controllable (as it puts developer in charge, not a compiler), it erases type details, it implies dynamic memory allocation and reference counting, it's slower and cannot be optimised by compiler. Compile-time capabilities in contrary were more interesting for the swift core team and the community as they are integrated into compiler and interact with other compile-time features.

Existential type is quite a simple language concept, so it wasn't even widely discussed until recently (not much to talk about). So most public discussions regarding protocols are being held in the context of generics. *(There is an opinion that Swift as "Protocol-oriented language" means more "Generics-oriented language"... but it doesn't sound as fancy)*.

Existential type is what protocols in Swift inherited from Objective-C as "default behaviour". You didn't need any additional syntax to define or return an existential type. In contrary the other features like generic types, when-condition, opaque types require some specific words or constructions. Eventually it started to look paradoxical that *major features* of protocols related to the generic types require more explicit syntax then *secondary feature* - existentials. The ideas how to tackle it had been discussed for quite some time before the changes even got to proposal stages.

But first things first...

### Filling in the gaps

To consider all the capabilities as one feature some visual gaps should have been fixed.

The biggest gap, as the Core team saw it, was the absence of the ability to return a generic type from a function. Existentials could be used both as parameters and result values. But on a compilation level you could only pass a generic type as a parameter, but were not able to describe the result value. That's how **opaque types** appeared (not without SwiftUI playing a noticeable role).

When opaque types were introduced and then expanded to more use cases, the next challenge was to minimize the interference between generic and existential types. Meaning that one shoul be possible to use in case of the other.

The idea of so called **generalized existentials** was already mentioned in [GenericsManifesto](https://github.com/apple/swift/blob/main/docs/GenericsManifesto.md#generalized-existentials). The essence was to make it possible to use generic-type protocols (the ones with associated type or self-constrained) as existential. Another challenge was to turn such existential back to generic when you need it. Some smaller potential improvements were needed for specific use cases. Some of the feature limitations were artificial (kind of legacy), the other required significant changes, but most of the goals were achieved. A number of discussions were held on forum followed by proposals, and eventually several feature changes were released into the language (see References).

As a result, mixing existentials and generic types became seamless. You can create a generic type protocol and use it as existential and vice-versa. The amount of related compilation issues drastically decreased. Now understanding the difference between runtime and compile-time cases is not needed in most of the cases... as it just work.


### Completing the paradigm shift

But one "small" change stands out of the list of improvements: **explicit existentials** (introducing `any`). It didn't add any new capabilities.

As time was passing by since the beginning, developers were more and more discouraged to use Existentials. Compile-time capabilities of protocols on the other hand were more and more extended and promoted. They became heavier in functionality and more valuable for the developers. Every other developer, when talking about protocols, kept enumerating the capabilities without even mentioning the original runtime polymorphism. It was just some small feature in the back of their minds, almost a nice side-effect of using protocols. 

People in the community (first of all, the Core team) kept questioning the status quo: having existentials as default was considered less and less acceptable. But you cannot just swap the syntax for two different language features, so it was decided to deprecate the default behaviour and create a special syntax for existentials.

That's why in case of existential declarations (see the code example before) we now should write:

```swift
var someAnimal: any Animal = Cat()
var animals: [any Animal] = [Cat(), Dog()]
```

I'm saying "should" but now it's a transitive period before the old syntax gets deprecated (expected in the next major language version). So we actually "have to" adopt this new way.

The change makes usage of existentials more explicit. Now it becomes a conscious choice rather than default option. It also eliminates some confusion when mixing existentials with compile-time constrains. 

But the most important thing here, imho, is the shift of the protocol paradigm, from legacy Objective-C like runtime interface to a big part of a compile-time abstraction. The shift started when protocols were introduced in Swift, but only now it seem to be completed. From now on protocols will be perceived differently, now there are no doubts that protocols are first of all made for generic or opaque types, associated values, where-conditions and so on, and runtime polymorphism officially takes the second (third, fourth) place.

What can we see in future?

Who knows, maybe eventually the default syntax (defining a value, parameter or result with a protocol without any additional words) will be reassigned back, but this time to the generic/opaque types. That would look like a logical completion of the shift. Possibly in a couple of major releases we will see such syntactic simplification as a new feature in Swift.

The other possible continuation of this story could be compiler warnings that advise you not to use existential when your use case can be covered by a generic/opaque type (Isn't it already there? Seems logical and easy to implement... but I haven't heard about it yet).

Even though we might see some other enchancements in this area, as well as further development for the compile-time capabilities of protocols, the shift of the paradigm is completed. Bye-bye dynamic interfaces!


## References

#### Main changes regarding opaque types:

- [[SE-0244] Opaque Result Types ](https://github.com/apple/swift-evolution/blob/main/proposals/0244-opaque-result-types.md)    
Introducing opaque return types. That's the feature that was introduced mainly for SwiftUI back in Swift 5.1
- [[SE-0328] Structural opaque result types](https://github.com/apple/swift-evolution/blob/main/proposals/0328-structural-opaque-result-types.md)	
Extending the use cases for the opaque return type. Now it can be a part of another result structure (a tuple or a closure)
- [[SE-0341] Opaque Parameter Declarations](https://github.com/apple/swift-evolution/blob/main/proposals/0341-opaque-parameters.md)    
An ability to use opaque types as parameters: more lightweight syntax for parameters with compile-time polymorphism.

#### Additional changes regarding to opaque types:

- [[SE-0346] Lightweight same-type requirements for primary associated types](https://github.com/apple/swift-evolution/blob/main/proposals/0346-light-weight-same-type-syntax.md)   
An ability to use more precise and complex compile-time parameters with more lightweight syntax.
- [[SE-0360] Opaque result types with limited availability](https://github.com/apple/swift-evolution/blob/main/proposals/0360-opaque-result-types-with-availability.md)    
More future-proof abstraction of opaque types. Now it allows you to return new types as previously declared opaque types. Capability specifically added for making APIs with opaque result types less breakable.

#### Main changes regarding existential types:

- [[SE-0309] Unlock existentials for all protocols](https://github.com/apple/swift-evolution/blob/main/proposals/0309-unlock-existential-types-for-all-protocols.md)  
Now generic type protocols can be used in some cases (still with limitations) as existentials.
- [[SE-0335] Introduce existential any](https://github.com/apple/swift-evolution/blob/main/proposals/0335-existential-any.md)    
Making usage of existencial types more explicit.
- [[SE-0352] Implicitly Opened Existentials](https://github.com/apple/swift-evolution/blob/main/proposals/0352-implicit-open-existentials.md)    
An ability to use existential types in some compile-time constrained cases (generics). 
- [[SE-0353] Constrained Existential Types](https://github.com/apple/swift-evolution/blob/main/proposals/0353-constrained-existential-types.md)   
Making it possible to use compile-time constrained protocol as existensials and still keep those constraints (being able to utilize them after)
- [[SE-0375] Opening existential arguments to optional parameters](https://github.com/apple/swift-evolution/blob/main/proposals/0375-opening-existential-optional.md)    
Allows an argument of (non-optional) existential type to be opened to be passed to an optional parameter. 

#### Key discussions at the forum:

- [Improving the UI of generics](https://forums.swift.org/t/improving-the-ui-of-generics/22814)
- [Lifting the “Self or associated type” constraint on existentials](https://forums.swift.org/t/lifting-the-self-or-associated-type-constraint-on-existentials/18025)
- [Improving the UI of generics](https://forums.swift.org/t/improving-the-ui-of-generics/22814)
- [Reverse generics](https://forums.swift.org/t/reverse-generics-and-opaque-result-types/21608)

#### Other relevant reading:

- [Swift Protocols: Magic of Dynamic & Static methods dispatches ✨](https://medium.com/@PavloShadov/https-medium-com-pavloshadov-swift-protocols-magic-of-dynamic-static-methods-dispatches-dfe0e0c85509)
- [What’s new in Swift 5.6](https://www.hackingwithswift.com/articles/247/whats-new-in-swift-5-6)
- [What’s new in Swift 5.7](https://www.hackingwithswift.com/articles/249/whats-new-in-swift-5-7)
- [Using the ‘some’ and ‘any’ keywords to reference generic protocols in Swift 5.7](https://www.swiftbysundell.com/articles/referencing-generic-protocols-with-some-and-any-keywords/)
