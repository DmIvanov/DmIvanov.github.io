---
title: Protocol extensions
date: 2020-09-22
author: topolog
layout: post
permalink: /protocol-extensions/
image:
  path: images-posts/2020-09-22-protocol-extensions/van-1920.jpg
  thumbnail: images-posts/2020-09-22-protocol-extensions/van-600.jpg
  caption: Photo by Adiwangsa Reinhart on Unsplash
tags:
  - iOS
  - swift
  - extensions
  - protocols
categories:
  - Tech Blog
share: true
---

When people speak about how powerful protocols are in Swift, in a lot of cases they consider protocol extensions as part of this power. That's unfair because it's a separate language feature, and an interesting one. Here we will detach and dissect it.

### Data type extensions

Before talking about the "protocol extensions" let's say a couple of words about the extensions in general (as a starting point).

Extensions is a powerful concept which was inherited from ObjC but there it's called "category" (there are also "extensions" in ObjC, but it's [a slightly different animal](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Chapters/ocCategories.html))

Extensions let you add some functionality to the data types you don't own. With the extension you can not only add new functions but also computed instance or type properties, provide new initializers or make a type to conform a protocol (more about that in the [official docs](https://docs.swift.org/swift-book/LanguageGuide/Extensions.html))

However, people also like to add extensions to their own types for reasons like grouping and separating the logic (some examples [here](https://www.natashatherobot.com/using-swift-extensions/) or [here](https://cocoacasts.com/four-clever-uses-of-swift-extensions)).

## Protocol extensions

Protocol extensions are different. You cannot "extend" a protocol because by definition a protocol doesn't have an implementation - so nothing to extend. _(You could say that we "extend a protocol WITH some functionality", but even an extended protocol is not something we can apply a function to.)_ Instead you still extend a data type, but through the conformance to a protocol.

```swift
protocol UITableViewDataSource { ... }

// extension to a data type
extension ViewController: UITableViewDataSource { ... }

// extension to a protocol
extension UITableViewDataSource { ... }
```

As well as protocols themselves, protocol extensions also have several faces. The difference may seem subtle, but in reality, it makes quite a big difference.

### Default implementation

First of all people use protocol extensions for default method implementation.

Let's say you have a protocol with couple of methods and some structs conforming to it:

```swift
protocol NewsProvider {
    func fetchNews() -> [News]
    func applyFilter(filter: Filter)
}

struct ReutersNewsProvider: NewsProvider {
    func fetchNews() -> [News] {
        // implementation
    }
    func applyFilter(filter: Filter) {
        // implementation
    }
}

struct AssociatedPressNewsProvider: NewsProvider {
    func fetchNews() -> [News] {
        // implementation
    }
    func applyFilter(filter: Filter) {
        // implementation
    }
}
```

Fetching is different for each news provider (different urls, data parsing,...) but filtering might be identical. Hence to avoid code duplication we move this functionality into the protocol extension:

```swift
protocol NewsProvider {
    func fetchNews() -> [News]
    func applyFilter(filter: Filter)
}

extension NewsProvider {
    func applyFilter(filter: Filter) {
        // implementation
    }
}

struct ReutersNewsProvider: NewsProvider {
    func fetchNews() -> [News] {
        // implementation
    }
}

struct AssociatedPressNewsProvider: NewsProvider {
    func fetchNews() -> [News] {
        // implementation
    }
}
```
Now all the data type which conform to `NewsProvider` get this functionality for free, without any duplication of the logic. In case you have some special news provider which needs to use some other filtering logic you can easily redefine it:

```swift
struct RussiaTodayNewsProvider: NewsProvider {
    func applyFilter(filter: Filter) {
        // specific implementation... cause you know...
        // these official Russian news agencies require some extra filtering    
    }
}
```

#### Optional protocol functions

You can also use protocol extensions to make optional functions in your protocol. Lets say some of the news providers offer a photo feed.

```swift
protocol NewsProvider {
    //required
    func fetchNews() -> [News]
    // optional
    func fetchPhotos() -> [Photo]?
}

extension NewsProvider {
    func fetchPhotos() -> [Photo]? {
        return nil
    }
}
```

That's still the same `default implementation` use case, but we provide empty implementation by default. And if the news provider is capable of fetching photos it overrides this function. So the specific implementation will be used instead of the default one.

In our client code, we use something like that to do something if there is some implementation.

```swift
if let photos = newsProvoder.fetchPhotos() {
    showPhotoFeed(with: photos)
}
```

### Additional functionality

Let's get back to `func applyFilter(filter: Filter)` which we moved from the concrete provider-structs to the NewsProvider-extension.

```swift
protocol NewsProvider {
    func fetchNews() -> [News]
}

extension NewsProvider {
    func applyFilter(filter: Filter) {
        // implementation
    }
}
```

At some point we might notice that there is no providers that override this method, so we can remove it from the protocol and just leave it in the extension. Everything will be working the same.

```swift
protocol NewsProvider {
    func fetchNews() -> [News]
}

extension NewsProvider {
    func applyFilter(filter: Filter) {
        // implementation
    }
}
```

Now it's not a `default implementation` but an `additional functionality` instead. The former is something that you can redefine in your object. But the latter is something that you get (for free) together with the protocol and you cannot do anything with it.

In this case protocol extension works like data type extension, but functionality is added in a more implicit way (more about it later).

Additional functionality in protocol extensions is exactly what lets us implement a data-type agnostic behavior. And then share it between several different data types. Each data type just declares the conformance to the protocol and gets the functionality from the protocol extension. That's something similar to [traits](https://en.wikipedia.org/wiki/Trait_(computer_programming)) and [mixins](https://en.wikipedia.org/wiki/Mixin).

### Static vs dynamic.

Why is it important to distinguish these two types of functions in protocol extensions?

The thing is that in case of the **default implementation** _dynamic method dispatch_ is being used. Whereas for the **additional functionality** it is _static method dispatch_. (If you need a deep dive into **method dispatch** in Swift read [this great article](https://www.rightpoint.com/rplabs/switch-method-dispatch-table). More details about this very difference in protocol extensions can be found [here](https://oleb.net/blog/2016/06/lily-ballard-swift-dispatch/).)

In practice it has some interesting consequences:
- additional functions are tightly linked to the function declaration (as far as the function is not in a protocol, there is no separate declaration at all)
- they exist in the heap even when the data type exists in the stack
- they are static, so they are faster and they can be optimized by the compiler
- they are not dynamic, so they cannot be overwritten in runtime

### To be continued

Here we discussed what the animal is "protocol extensions". We mentioned 2 types of functions and the general differences between them. In the [next article](/pitfalls-of-protocol-extensions/) we are diving more into practical cases and pitfalls regarding using this feature in our code. Stay tuned ;-).
