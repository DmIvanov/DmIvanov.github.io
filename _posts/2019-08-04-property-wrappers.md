---
id: 368
title: Property Wrappers
date: 2019-08-04T08:30:28-02:00
author: topolog
layout: post
guid: https://dmtopolog.com/?p=368
permalink: /property-wrappers/
image:
  path: ⁨images-posts/2019-08-04-property-wrappers/boxes.jpg
  thumbnail: images-posts/2019-08-04-property-wrappers/boxes-768x513.jpg
categories:
  - Tech Blog
tags:
  - language feature
  - swift
---
In my opinion property wrappers are one of the most interesting features of SwiftUI and the upcoming Swift 5.1. Here I&#8217;ll share what has caught my attention and what looks the most interesting about this brand new thingy.

<div style="height:100px" aria-hidden="true" class="wp-block-spacer">
</div>

## The story of the feature.

Nobody argues that Apple is really good at presenting products on stage (devices, tools, language feature). They know how to describe the thing in couple of sentences and expose it in a most compelling way. And the entire description looks very lightweight which some times can confuse even experienced developers.

Right after WWDC&#8217;19 some developers (myself including) discussed that finally Apple reacted on such a trendy things like declarative interfaces, reactive paradigm and unidirectional architecture. And only days after some bits and pieces of information emerged and it got clear that the work was ongoing for years before the first release in June&#8217;19.

The same happened with property wrappers. The first time the idea about what now is known as `Property Wrappers` was presented in the end of 2015 as [a mail-thread discussion](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151214/003148.html). [Proposal #30](https://github.com/apple/swift-evolution/blob/master/proposals/0030-property-behavior-decls.md) in the new shiny swift repository followed soon after. In that proposal the feature was originally called `Property Behaviors`. At that time Swift was just open sourced, the early adopters were busy migrating their code bases from Swift 1 to Swift 2 and writing JSON parsers and we had to write `UIColor.blueColor()` and `dispatch_queue_create("com.test.myqueue", nil)`.

At that moment Core Team already realised that new property patterns like `lazy` or `NSCopying` kept appearing again and again and maybe it&#8217;s not the best solution to keep on adding new modifiers for them into the compiler. That made the compiler logic more and more complex and developers more and more confused. Moreover iOS programmers had to frequently copy-past some `getter/setter` logic from model to model which also wasn&#8217;t good for the code they wrote. That was the main reasoning behind the proposal.

The usage of such a property in case of `lazy` would have looked as follows:

<pre class="wp-block-code"><code>var [lazy] foo = 1738</code></pre>

But with all the appreciation the proposal was eventually rejected ([the rationale](https://lists.swift.org/pipermail/swift-evolution-announce/2016-February/000047.html)). Swift language was too changing and not stable enough at that point so implementing this functionality could cause additional complications. The feature should have being changed together with rapidly evolving language. But the work on the subject haven&#8217;t stopped.

The next time the discussion emerged at the Swift forum in March&#8217;19 ([the pitch](https://forums.swift.org/t/pitch-property-delegates/21895)). This time the feature was called `Property Delegates`, the essence and motivation remained the same but its syntax was refined under the inspiration of [Delegated Properties in Kotlin](https://kotlinlang.org/docs/reference/delegated-properties.html).

Our beloved `lazy` declaration would have looked like:

<pre class="wp-block-code"><code>var foo by Lazy = 1738</code></pre>

[Proposal #258](https://github.com/apple/swift-evolution/blob/master/proposals/0258-property-wrappers.md) followed that pitch in the end of May&#8217;19. The name and the syntax were changed one more time and in the end we got what is a part of Swift 5.1 with usage like:

<pre class="wp-block-code"><code>@Lazy var foo = 1738</code></pre>

I see this small piece of Swift history as a good example of how the features are being added to the language. Sometimes the feature is not ready enough to be added (you can see it in number of other proposals), sometimes it might not make sense at all, or might be rejected by the community. In another case the feature is ready and appreciated but the language is not mature enough, or not in the proper condition to absorb it. Even if eventually they meant to be one whole thing (a new version of the language) there should be &#8220;a match&#8221; like in a dating app for them to get together.

<div style="height:100px" aria-hidden="true" class="wp-block-spacer">
</div>

## A brief essence.

Property wrappers are among that things which are easy to describe but not so easy to understand until you use them in practice. Here I&#8217;ll not give you any practical examples as there already are lot&#8217;s of them around. For instance you may check out this [great post on NSHipster](https://nshipster.com/propertywrapper/) or this [GitHub repo](https://github.com/guillermomuntaner/Burritos).

In two words it&#8217;s a generalisation of a pattern, another layer of abstraction (what can be better than one more level of abstraction!). I think most of you know the general idea, but let me briefly remind it.

Imagine you have an object&#8217;s property and some additional functionality you want to apply when the property&#8217;s value is being set of accessed. Something like lazy loading, some data modifications or additional actions. Most of these things are something that you would usually do in property&#8217;s accessors (set/get, didSet/didGet, willSet/willGet). Property wrappers abstract these actions from the specific data types and make it applicable to the other data models. You basically take the code to implement it and move it to a separate place which makes possible to reuse it.

<div style="height:100px" aria-hidden="true" class="wp-block-spacer">
</div>

## Additional functionality.

In a sense property wrappers is an implementation of [the decorator pattern](https://en.wikipedia.org/wiki/Decorator_pattern), but this &#8220;decoration&#8221; is restricted to the accessors of this data structure.

<div class="wp-block-image">
  <figure class="aligncenter"><img src="https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/08/client-wrapper-wrapped-1.png?resize=688%2C167&#038;ssl=1" alt="" class="wp-image-385" srcset="https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/08/client-wrapper-wrapped-1.png?resize=1024%2C248&ssl=1 1024w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/08/client-wrapper-wrapped-1.png?resize=300%2C73&ssl=1 300w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/08/client-wrapper-wrapped-1.png?resize=768%2C186&ssl=1 768w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/08/client-wrapper-wrapped-1.png?w=1473&ssl=1 1473w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/08/client-wrapper-wrapped-1.png?w=1376&ssl=1 1376w" sizes="(max-width: 688px) 100vw, 688px" data-recalc-dims="1" /></figure>
</div>

Property wrappers can also be treated as a way to add some functionality without changing the object&#8217;s original implementation. Consider `extensions` (categories and extensions to the classes in ObjC, protocol extensions in swift), it is the mechanism to add new methods (and properties &#8211; if we don&#8217;t mind using some variables in the heap and a bit of runtime magic) to existing data structures. Inside an extension you can refer to the object&#8217;s properties and accessible methods.

Property wrapper looks a bit like an extension &#8211; it also extend the existing behaviour &#8211; but it knows nothing about the object it wraps and its structure (properties, methods). Instead it knows about the lifecycle (when the value is being set or get) so it lets us to do something in between the object and the code using it in the moments of data accessing. Property wrapper is a man-in-the-middle, a proxy object which existence is hidden from the wrapped structure as well as from the client code. Of course we ad a modifier like `@Modified` next to the property declaration but that&#8217;s it. No other modifications required for the existing code of both the wrapped object and the client code.

Property wrapper adds new behaviour to a specific instances of the data structure. It doesn&#8217;t influence all the other instances of this type (in a contract to the extension or subclassing). So changing the functionality of just one property, one instance of some data structure you shouldn&#8217;t worry about any side effects for the others.

<div style="height:100px" aria-hidden="true" class="wp-block-spacer">
</div>

## The layers

Let&#8217;s take a look at our projects considering the levels of code abstractions.

<div class="wp-block-image">
  <figure class="aligncenter"><img src="https://i0.wp.com/dmtopolog.com/wp-content/uploads/2019/08/layers-1.png?resize=688%2C381&#038;ssl=1" alt="" class="wp-image-384" srcset="https://i0.wp.com/dmtopolog.com/wp-content/uploads/2019/08/layers-1.png?resize=1024%2C567&ssl=1 1024w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2019/08/layers-1.png?resize=300%2C166&ssl=1 300w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2019/08/layers-1.png?resize=768%2C425&ssl=1 768w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2019/08/layers-1.png?w=1143&ssl=1 1143w" sizes="(max-width: 688px) 100vw, 688px" data-recalc-dims="1" /></figure>
</div>

The code we write (let&#8217;s call it business Logic) depends on variety of libraries and frameworks. Some of them are system ones, some are our own, the others are third party. Libraries and frameworks (as well as our own code) use the language with its syntax and type system. Somewhere in between the code and the language we can distinguish the standard library (`stdlib`) which is usually considered as a part of a language by us developers, but the language maintainers see it as something external ([Swift Standard Library](https://developer.apple.com/documentation/swift/swift_standard_library)). We can go deeper into the language and talk about the compiler, its optimisations and translation our code through some intermediate language right into the binary code (I tackled it a bit in [one of my previous posts](https://dmtopolog.com/code-optimization-for-swift-and-objective-c)), but it&#8217;s out of the scope of our topic.

Each layer has its functionality and features. The language gives us its type system, operators, syntactic constructions and all the other basic things. In case of swift standard library adds fundamental data types (Int, String), common data structures (Array, Dictionary, Set), some basic global functions and protocols. Then frameworks add some domain specific blocks of functionality which we use in our business logic.

Each layer has its context and area of influence. The language has the widest context and it influences all the code (frameworks, libs, projects, utils) written on this language. So changes on a language level are the most important and the most dangerous. The business logic of a specific application has the narrowest context and influences nothing but one project.

Between the layers there are APIs. The language is kind of API on its own, the stdlib has its API, all the libs and frameworks have their API.

There is a general rule in programming (and in designing the APIs) that you should minimise the context (the visibility) for all the features you add. If you don&#8217;t need a constant to be global make it local; if you don&#8217;t explicitly need a public function make it private; if you don&#8217;t need a class to be subclassed make it final. All this simple heuristics push you to set as minimum context for the functionality as possible. The lesser the context is the easier to control the change, the more predictable is the overall system&#8217;s behaviour, the less side effects it brings. It&#8217;s also easier to extend the API, to broaden the context than to make it smaller. So the subsystem which operates in the smallest context is more future-proof. Modularity and microservices also come from this rule.

And here come property wrappers. The feature minimises the context of the potential change. If you can add the functionality at the framework/business logic level instead of the language (as it was before with `lazy` or `NSCopying`), that&#8217;s the way to go.

<div style="height:100px" aria-hidden="true" class="wp-block-spacer">
</div>

## Simplifying the compiler logic

The more features are integrated into the language itself the bigger is the &#8220;set of rules&#8221; the compiler uses to translate our code into the binary data. The more job for the compiler the more time it needs to create a binary program. Too much of different language features may also cause clashes on the compilation stage and errors with vague description. So there should always be some balance for the language and its compiler between being useful/helpful and being cumbersome/misleading.

As the original proposal says, property wrappers meant to simplify the compiler logic (among other reasons). Better said they should prevent the over-complication of it in future. It doesn&#8217;t seem like they remove `lazy` from the language as it can be easily implemented now with property wrappers. But swift maintainers gave the developers the tool to implement something similar without adding new features to the language, without adding new rules to the compiler, without making the compiler slower and more complex. So instead of filling in proposals and chasing the maintainers for adding a new feature, now in some cases you can just add this functionality on a level you have access yourself.

<div style="height:100px" aria-hidden="true" class="wp-block-spacer">
</div>

## Syntax and aesthetics

Before swift 5.1 if you see `@` in swift code you know that you deal with some language feature. Getting back to the layers, `@` meant something belonging to the language or stdlib layer. Although the amount of such modifiers has been growing quite fast, we still could remember the majority of such keywords. Property wrappers now let us to introduce our own `@`-prefixed constructions. Hence we are losing this clear border between the language features and our third party code. When moving the burden from the compiler to our code we blur the edge between these worlds. Making the language and the compiler simpler we make the overall code base more complex.

So now in code you may see something like this:

<pre class="wp-block-code"><code>class Dessert {
    @DecoratedWithChocolate @Diced @Sliced var banana: Banana
}</code></pre>

Functional? Flexible? Fancy? Yep. Readable?… I&#8217;m not sure. That&#8217;s subjective. I think at some point we all will get used to it. Until then I share the concern that all these additional `@`s make the code less structural and a bit more messy. The edges between the API layers become blurred. This steeper learning curve is the price we have to pay for the ability to use this construction.

The readability suffers especially considering the parameters you can add to the wrapper:

<pre class="wp-block-code"><code>@UserDefault("test", defaultValue: "Hello, World!", userDefaults: userDefaults)
var test: String</code></pre>

<div class="wp-block-image">
  <figure class="aligncenter"><img src="https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/08/Screenshot-2019-08-04-at-12.03.15.png?fit=688%2C370&ssl=1" alt="" class="wp-image-371" srcset="https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/08/Screenshot-2019-08-04-at-12.03.15.png?w=1180&ssl=1 1180w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/08/Screenshot-2019-08-04-at-12.03.15.png?resize=300%2C161&ssl=1 300w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/08/Screenshot-2019-08-04-at-12.03.15.png?resize=768%2C413&ssl=1 768w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/08/Screenshot-2019-08-04-at-12.03.15.png?resize=1024%2C550&ssl=1 1024w" sizes="(max-width: 688px) 100vw, 688px" /><figcaption><br /><a href="https://twitter.com/nataliya_bg/status/1140519869361926144">https://twitter.com/nataliya_bg/status/1140519869361926144</a></figcaption></figure>
</div>

<div style="height:100px" aria-hidden="true" class="wp-block-spacer">
</div>

## The chicken or the egg?

There are some thoughts around about how much SwiftUI dictated the evolution of swift. Several features were rushed to be included into the language before Swift 5 release. Some of them were still not release on time (mostly because swift evolution process is not completely controlled by Apple), so when SwiftUI was presented they have not yet been parts of the language. So they were planned for Swift 5.1 and SwiftUI had to use them internally other then the built-in language features. These features being a general improvements to the language eventually empowered SwiftUI and the whole picture appeared from the separate pieces.

The question what came first: property wrappers or SwiftUI may seem a bit naive. We see that there is quite a story behind this language feature and the (official) original motivation have nothing to do with SwiftUI. Moreover the discussion was started long before SwiftUI… But was it? There are also signs, evidences and replies from the members of the Swift Core Team that SwiftUI was under development years before the release at WWDC&#8217;19. Some even say the first research was started before Swift was released in 2014. So who knows whether or not we would have had property wrappers in the language if not SwiftUI.
