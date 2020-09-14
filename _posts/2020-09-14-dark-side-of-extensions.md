---
title: Dark side of extensions in Swift
date: 2020-09-14
author: topolog
layout: post
permalink: /dark-side-of-extensions/
image:
  path: images-posts/2020-09-14-dark-side-of-extensions/moon-1920.jpg
  thumbnail: images-posts/2020-09-14-dark-side-of-extensions/moon-600.jpg
  caption: Photo by Andrew Hughes on Unsplash
tags:
  - iOS
  - swift
  - extensions
categories:
  - Tech Blog
share: true
---


Extension is a very powerful concept in Swift. It has several different applications in our code: we can extend the data types we don’t own, we can extend our own ones, we can separate the functionality and protocol conformance. Each one of them gives us additional capabilities and extra powers, but some of them come together with drawbacks which we have to consider.


## Extensions are implicit

First of all, everything I say in this section is very subjective. Matters like "readability", "clearness" and "explicitness" are different for everybody.

The idea of the extension as a concept is that you just add some functionality `X` to the object `Y`. This object `Y` is already a piece of some complete encapsulated logic. But we want more. For some reason, we decide to structure this additional logic to look like a part of the object `Y`.

Sometimes it is indeed some functionality which logically (another subjective term) belongs to the entity you extend. So when you see `Y.X()` in code you understand clearly what it means "to do `X` with `Y`".

Consider `shuffle()` for the array or `toggle()` for bool _(both of these functions were not parts of a standard library in the early days, so lots of folks had them in the extensions)_. There are some good examples from Paul Hudson [here](https://www.hackingwithswift.com/articles/141/8-useful-swift-extensions) or John Sundell [here](https://www.swiftbysundell.com/articles/writing-reusable-swift-extensions/).

When you see something like that it's pretty self-explanatory:

```swift
let phrase = "The rain in Spain"
print(phrase.wordCount)

let items = Bundle.main.decode([TourItem].self, from: "Tour.json")

article.cacheOnDisk()
```

In other situations that additional functionality `X` semantically might not look like it should belong to the object `Y`. So `Y.X()` doesn't give you the full context when you see it in code. _(It's fair not only for extensions but for regular object functions as well. But with extensions it's much easier to add something which shouldn't belong to the extended entity)_

For instance, imagine somebody added a smart extension to `Double`, you have no idea about it and just found such code:

```swift
let length = 25.4.mm

let aMarathon = 42.km + 195.m
```

What do you think are the units of these constants? Is `length` in millimeters? What about `aMarathon`? How can you further use them? Is it some value for UI, or something to encode and pass to the backend?

To answer these questions and be able to use such extensions in code you need to check the implementation, because there is some missing (implicit) logic here:

```swift
extension Double {
    var km: Double { return self * 1_000.0 }
    var m: Double { return self }
    var cm: Double { return self / 100.0 }
    var mm: Double { return self / 1_000.0 }
    var ft: Double { return self / 3.28084 }
}
```

What we miss is "conversion to meters". You cannot get this information from the name of the function.

And if you don't use this extension every day, quite possible you might need to do it next time you see such code.. What was that meters or centimeters? You might even add a quick-help comment to fill in this logical gap and save yourself some time in the future:

```swift
/**
 Conversion to METERS
*/
extension Double {
    //...
}
```

...but you still need to jump to a definition every time you are not sure. There is a piece of implicit logic here.

IMHO, that's not the best way to use extensions... even though that's what Apple suggests in the [official documentation](https://docs.swift.org/swift-book/LanguageGuide/Extensions.html#ID152)

It would be more explicit to create some primitive converter or a global static conversion function. Which option is more comprehensible?

```swift
let aMarathon = 42.km + 195.m

let aMarathon = Converter.kmToMeters(42) + 195
```

## Illusory separation

When you introduce an extension you consider the boundary between the base implementation and the extension. In the case of extending the objects, you don't own this boundary is quite physical. But if you own the base implementation and it's even in the same module, the boundary gets quite weak.

If the boundary is weak it tends to disappear. _(Partly I tried to explain it [here](https://dmtopolog.com/modularity-1-boundaries/))_ The entropy of every system tends to increase. Like two gases or two liquids of the same density, two pieces of logic tend to mix without a physical boundary.

You might use extensions [to group separate pieces of functionality](https://www.natashatherobot.com/using-swift-extensions/). In case of separating protocol conformance or a private/public logic the boundary is always clear. But sometimes it's not clear how to assign something to a group or a category.

Let's say we have a blog post data model and we prefer to separate all the persistence-related logic in an extension:

```swift
struct BlogPost {
    // general logic
}

// Persistence
extension BlogPost {
    init(path: String) {
        // implementation
    }
    func write(path: String) {
        // implementation
    }
}
```

But at some point we got validation logic which should be applied to the post before it's being written on disk. On one side, it's used exclusively for writing on disk. On another side, it's not directly related to working with disk. Should we put it to the base implementation or to the extension?

Another confusion is with stored properties. We cannot add them to extensions. So when we have some extension-related properties we have to keep them outside of the extension - in the main data-type implementation.

Let's say we want to add a directory path to our Persistence-extension, so we don't need to pass the full path from the outside.

```swift
struct BlogPost {
    let directoryPath = "path/to/a/directory/"
    // general logic
}

// Persistence
extension BlogPost {
    init(for id: String) {
        let path = directoryPath + id
        // implementation
    }
    func write() {
        let path = directoryPath + self.id
        // implementation
    }
}
```

It kinda spoils the entire idea of logical separation, because now we have a piece of extension-related logic in the main implementation.

_(ObjC runtime - the only alternative - may bring us more additional headache here, so it doesn't worth to be used just to compensate this logic-separation flaw, imho)_

If you add extensions to your own objects it might get even worse if you (or somebody from your team) start to use the extension functionality in the object's implementation:

```swift
class Foo {

    // main functionality

    func anotherFunction() {
        function()
    }
}

extension Foo {
    func function() {
        //...
    }
}
```

Sounds borderline impossible with a simple example. Why would one do that? Nonsense. But in a big project you might not remember all the layer/logic separation and what functionality belongs to which layer (to the object or to its extension). So when you implement `anotherFunction()` helpful autocomplete might suggest you to use `function()` and it might be exactly what you need.

As a result, your encapsulation is totally broken. Not only your extension relies on the base implementation (which kinda OK), but the other way around as well. These two pieces of logic get coupled too tightly. With extensions to your own objects, the boundary between them is quite vague by definition. But such interconnections make it just a formality.


## SRP violation

Single responsibility principle (SRP) pushes us to divide our code (project, module, class) into a number of separate logical units. Each unit should have only one (in practice, as least as possible) responsibility(-ies).

An extension is not a separate logical entity. Its functionality logically belongs to its base data type. So even when you put an extension to a separate file or the base type resides in a different module, for the client code there is still no boundary at all.

So in fact all the functionality you add via the extensions are additional responsibilities of your base type.

Guess who has a physical boundary? A separate object does! Hence if you create a separate object instead of the extension in the majority of cases you will write the same amount of reusable code. But architectural-wise it will be cleaner.

Instead of having some generic persistence-extension to you data models you just wrap the same functionality into a `PersistenceHelper` (don't even start to nag about the naming!), instead of caching extension - `CachingHelper`, instead of converting-extension - `Converter`, instead of the one which builds a network request out of the base model - separate `RequestBuilder`-object,... and so on.

I strongly believe that lots of small classes with narrow responsibilities are better than one fat object which knows how to do everything. Even if this fat object is logically perfectly separated between several files or modules.


## But look at the standard library!

In swift standard library there are plenty of extensions. People like to use it as proof that you should use as many extensions as possible. So if the swift core team doesn't seem to care about all the drawbacks I mentioned here, why the others should.

First, let's mention it again: these possible issues are the dark side of this language concept. And there is a bright one as well... maybe even several. So that's the matter of balance and priority.

Second, the standard library is on a different level of abstraction than the code you write (in majority of cases). We create and use objects of a much higher grade of abstraction, which contain more context and which are more complex entities. The standard library has no layers of abstraction underneath - just the language itself. It is a much more generic thing which suppose to satisfy lots of different use cases.

We shouldn’t blindly copy everything that Apple does. What works there is not necessarily good for our projects (and the other way around).


## Conclusion

> With great power comes great responsibility
(Peter Parker... [or some Frenchman from XVIII century](https://en.wikipedia.org/wiki/With_great_power_comes_great_responsibility))

I'm not talking you out of using extensions. I just want you to keep in mind all that nuances when using this language feature.

We didn't mention protocol extensions here, because that's a different topic. Hope to cover it in next posts, stay tuned!
