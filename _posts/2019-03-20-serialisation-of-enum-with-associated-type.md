---
id: 286
title: Serialization of enum with associated type
date: 2019-03-20T04:48:58-02:00
author: topolog
layout: post
guid: https://dmtopolog.com/?p=286
permalink: /serialisation-of-enum-with-associated-type/
image:
  path: ⁨images-posts/2019-03-20-serialisation-of-enum-with-associated-type/IMGP3601.jpg
  thumbnail: images-posts/2019-03-20-serialisation-of-enum-with-associated-type/IMGP3601-768x511.jpg
categories:
  - Tech Blog
tags:
  - enum
  - iOS
  - Serialization
  - swift
---
Enumerations are not just first-class citizens in Swift. They adopt many features traditionally supported only by classes, such as computed properties or static and instance methods. Enumerations can also define custom initialisers, can be extended to expand their functionality beyond their original implementation, and can conform to protocols to provide standard functionality.

Ability to have some associated data is one of the greatest enum features in Swift. (The notion itself has quite a long history in computer science and is being used in other languages &#8211; <https://en.wikipedia.org/wiki/Tagged_union>). That means that you can define an enumeration to store some associated values of any type alongside the actual cases. Moreover the amount and the types of these values can differ from case to case.

The classical example of such model is product barcodes. Imagine a system where you operate products marked by one of the two types of barcodes: 1D barcodes in UPC format and 2D barcodes in QR code format:

<div class="wp-block-image">
  <figure class="aligncenter"><img src="https://i0.wp.com/dmtopolog.com/wp-content/uploads/2019/03/bar-codes.png?w=688&#038;ssl=1" alt="" class="wp-image-289" srcset="https://i0.wp.com/dmtopolog.com/wp-content/uploads/2019/03/bar-codes.png?w=1004&ssl=1 1004w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2019/03/bar-codes.png?resize=300%2C134&ssl=1 300w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2019/03/bar-codes.png?resize=768%2C344&ssl=1 768w" sizes="(max-width: 688px) 100vw, 688px" data-recalc-dims="1" /></figure>
</div>

The former contains a sequence of 4 digits, the latter incapsulates a string. So for your inventory tracking system the following enum is an ideal data model or this object:

<pre class="wp-block-code"><code>enum Barcode {
    case upc(Int, Int, Int, Int)
    case qrCode(String)
}</code></pre>

It’s eventually quite convenient to use such data type in your code:

<pre class="wp-block-code"><code>var productCode = Barcode.upc(0, 36000, 29145, 2)
productCode = .qrCode("ABCDEFGHIJKLMNOP")

switch productCode {
case .upc(let numberSystem, let manufacturer, let product, let check):
    print("UPC: \(numberSystem), \(manufacturer), \(product), \(check).")
case .qrCode(let productCode):
    print("QR code: \(productCode).")
}</code></pre>

But what if at some point you need to store an array of such codes on disc or to send it to the backend? You need to somehow serialise the array. Bur how would you do that?

One might think, they can just declare a Codable conformance and it just works. Try it and you’ll see such an error:

<div class="wp-block-image">
  <figure class="aligncenter"><img src="https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/03/Error_DoesntConformCodable.png?w=688&#038;ssl=1" alt="" class="wp-image-291" srcset="https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/03/Error_DoesntConformCodable.png?w=779&ssl=1 779w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/03/Error_DoesntConformCodable.png?resize=300%2C50&ssl=1 300w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/03/Error_DoesntConformCodable.png?resize=768%2C127&ssl=1 768w" sizes="(max-width: 688px) 100vw, 688px" data-recalc-dims="1" /></figure>
</div>

Can’t the compiler infer the conformance? Do we need to implement Codable methods ourselves? The questions are not so trivial.

Let’s make a step back and take a look at more trivial enum type:

<pre class="wp-block-code"><code>enum Planet {
    case mercury, venus, earth, mars, jupiter, saturn, uranus, neptune
}</code></pre>

For such a simple one we can just add a raw type (`Int` or `String`) to the definition:

<pre class="wp-block-code"><code>enum Planet: Int {
    case mercury = 1, venus, earth, mars, jupiter, saturn, uranus, neptune
}</code></pre>

Making this change we create a simple way of serialising our data model using some primitive values which can be mapped into our enum cases. Now the enum case can be easily stored to UserDefaults or passed to the backend as a part of JSON file:

<pre class="wp-block-code"><code>let nextPlanetToVisit = Planet.mars
UserDefaults.standard.set(nextPlanetToVisit.rawValue, forKey: kStoredNextPlanet)

if let rawValue = try? JSONDecoder().decode(Int.self, from: jsonData) {
    let planetFromBackend = Planet(rawValue: rawValue)
}</code></pre>

But if we have an enum with associated types we cannot use this approach.

It should be clear that when we add some associated type to our enum we add another level of complexity to the data structure. Each enum case now cannot be easily represented by one value of a primitive type without losing some information, you broke that one-to-one relation. How can you represent `Barcode.upc(0, 36000, 29145, 2)` with just one primitive value so you can easily switch back and forth between the custom and the raw values. So an enum can have either a raw type or some associated values.

Here we are back to Codable. It was specifically created to make possible this serialisation of swift types like \`struct\` and \`enum\`. (I was discussing it in details [here](https://dmtopolog.com/object-serialization-in-ios/))

As most of you know, for adopting Codable the type has to implement these two methods:

<pre class="wp-block-code"><code>init(from decoder: Decoder) throws
func encode(to encoder: Encoder) throws</code></pre>

In plenty of cases swift compiler can generate an implementation of them itself so you don’t need to bother. For example, if we have an enumeration with a raw type, the compiler just infers implementation of these methods from the raw type. So that would be enough:

<pre class="wp-block-code"><code>enum Planet: Int, Codable {
    case mercury = 1, venus, earth, mars, jupiter, saturn, uranus, neptune
}</code></pre>

Just keep in mind that you cannot use JSONEncoder or PropertyListEncoder to serialise this enum. If you do you will catch _&#8220;Top-level Planet encoded as number property list fragment.”-_error&nbsp; The reason is that this two Swift serialisers use old JSONSerialisation/PropertyListSerialization classes from Foundation under the hood which can work only with some containers like arrays and dictionaries as root objects. So even though our enum is 100% codable it cannot be a root object of the graph. This considered as a bug (or something that is intended but simply not implemented yet in Swift) and the swift team intends to fix it in some of the future releases (Swift 5.x) (more about it here &#8211; <https://forums.swift.org/t/top-level-t-self-encoded-as-number-json-fragment/11001>)

So to serialise such simple enum you have several options:

  1. You can wrap it into array or dictionary (dirty but simple)
  2. You can use NSKeyedArchiver for serialisation. This guy doesn’t have that container restriction.
  3. You can provide custom implementation of Codable, which lets you easily containerise the value&nbsp;

The first two alternatives are quite clear so let’s talk then about this third option. As we aim to handle Codable implementation ourselves we don’t need our enum to have a raw type. If it does it’s not a problem, it just doesn’t play any role in serialisation.

Another brief insight in how Codable works in general. We deal with key-value container used by Encoders/Decoders. When encoding an object we put all the properties to the container and then this container is being serialised to the binary data. Inside `encode(to encoder: Encoder)` method we set the mapping table of how exactly our object’s properties should be packed into the container. When we decode an object in our `init(from decoder: Decoder)` we set the rules of translating the data from the container back into our model’s structure.

Say we want to implement the Codable conformance for a trivial enum without any associated values:

<pre class="wp-block-code"><code>enum Barcode {
    case upc, qrCode
}</code></pre>

_(Let’s imagine we cannot add a raw type)_

For encoding/decoding we will need coding keys for the key-value container. For this purpose we use nested enum with a special CodingKey type as a raw type.

<pre class="wp-block-code"><code>enum Barcode: Codable {
    case upc, qrCode
    enum CodingKeys: CodingKey {
        // ...
    }
} </code></pre>

Basically for encoding this enum we have 2 options (for simplicity we will discuss only encoding method). Firstly we can use just one key and encode different enum cases as different values for this key:

<pre class="wp-block-code"><code>    enum CodingKeys: CodingKey {
        case rawValue
    }

    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: Key.self)
        switch self {
        case .upc:
            try container.encode(0, forKey: .rawValue)
        case .qrCode:
            try container.encode(1, forKey: .rawValue)
        }
    }</code></pre>

So if we serialise both values with JSONEncoder and print the result we will see:

<pre class="wp-block-code"><code>let serialisedValue1 = try! JSONEncoder().encode(Barcode.upc)
let serialisedValue2 = try! JSONEncoder().encode(Barcode.qrCode)
print(String(data: serialisedValue1, encoding: .utf8)!)
print(String(data: serialisedValue2, encoding: .utf8)!)</code></pre>

**{&#8220;rawValue&#8221;:0}**  
**{&#8220;rawValue&#8221;:1}**

Decoding is quite straightforward with such approach: we check the value for our key and then switch to the corresponding enum case. I called the key “rawValue” because that’s actually the same approach that is used when dealing with enums with raw type.

The second approach is to use different keys for different enum cases:

<pre class="wp-block-code"><code>    enum CodingKeys: CodingKey {
        case upc, qrCode
    }

    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        switch self {
        case .upc:
            try container.encode(true, forKey: .upc)
        case .qrCode:
            try container.encode(true, forKey: .qrCode)
        }
    }</code></pre>

If we print serialised data here we will see the following:

**{&#8220;upc&#8221;:true}**  
**{&#8220;qrCode&#8221;:true}**

The value we store here is not important at all. When we decode the enum from the container we check all the possible keys one after another until we find some value for requested key.

<pre class="wp-block-code"><code>    init(from decoder: Decoder) throws {
        let values = try decoder.container(keyedBy: CodingKeys.self)
        if let _ = try? values.decode(Bool.self, forKey: .upc) {
            self = .upc
            return
        } else if let _ = try? values.decode(Bool.self, forKey: .qrCode) {
            self = .qrCode
            return
        } else {
            throw EncodingError.dataCorrupted
        }
    }</code></pre>

Although it might seem as less efficient approach comparing to the previous one, it’s more convenient in case of associated values in our enum cases.

If we are sure that each enum case might have not more than one associated value (no mater which type) we can still use the last approach we discussed:

<pre class="wp-block-code"><code>enum Barcode: Codable {
    case upc(Int)
    case qrCode(String)

    enum CodingKeys: CodingKey {
        case upc, qrCode
    }

    init(from decoder: Decoder) throws {
        // ...
    }

    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        switch self {
        case .upc(let value):
            try container.encode(value, forKey: .upc)
        case .qrCode(let value):
            try container.encode(value, forKey: .qrCode)
        }
    }
}</code></pre>

If we serialise such enum into JSON and print the cases out we will se the date in the following format:

**{&#8220;upc&#8221;:29145}**  
**{&#8220;qrCode&#8221;:&#8221;ABCDEF&#8221;}**

Here we still have the actual enum case as a key and we put our associated value as the value for this key. When decoding the object from the container we know which case has which associated type so it’s not a problem to properly decode it.

<pre class="wp-block-code"><code>    init(from decoder: Decoder) throws {
        let values = try decoder.container(keyedBy: CodingKeys.self)
        if let value = try? values.decode(Int.self, forKey: .upc) {
            self = .upc(value)
            return
        } else if let value = try? values.decode(String.self, forKey: .qrCode) {
            self = .qrCode(value)
            return
        } else {
            throw EncodingError.dataCorrupted
        }
    }</code></pre>

But it’s clear that we have to think about something smarter when we have more than one associated value per enum case, as we cannot normally store several values under one case-related key. Things get even more interesting if we have different amount of associated values for each case.

Let’s take another example. In my experience such enums with associated value perfectly fit the use case of some complicated action consisting of several heterogeneous steps like an on-boarding, downloading or updating some system. It might be handy to persist the state so we can start from the place the process was previously interrupted.

Imagine we have an update process which consist of these 3 steps:

<pre class="wp-block-code"><code>enum UpdateState {
    case started
    case storedToFile(path: String)
    case uploadingToDevice(fromPath: String, percentage: Int)
}</code></pre>

Some of them have associated values, others don’t. You can think about different ways of implementing that.&nbsp;

For instance we can create coding keys per case and come up with some general wrappers for different amount of associated values:

<pre class="wp-block-code"><code>    enum CodingKeys: CodingKey {
        case started
        case storedToFile
        case uploadingToDevice
    }

    struct NoValue: Codable {}

    struct OneValue&lt;T: Codable>: Codable {
        let value: T
    }

    struct TwoValues&lt;T1: Codable, T2: Codable>: Codable {
        let value1: T1
        let value2: T2
    }</code></pre>

If we want to use the same structure for different cases we should pick quite general names. Our encoding will look like:

<pre class="wp-block-code"><code>    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        switch self {
        case .started:
            try container.encode(NoValue(), forKey: .started)
        case .storedToFile(let path):
            try container.encode(OneValue(value: path), forKey: .storedToFile)
        case .uploadingToDevice(let fromPath, let percentage):
            try container.encode(TwoValues(value1: fromPath, value2: percentage), forKey: .uploadingToDevice)
        }
    }</code></pre>

Looks quite neat. Decoding is also quite simple:

<pre class="wp-block-code"><code>    init(from decoder: Decoder) throws {
        let values = try decoder.container(keyedBy: CodingKeys.self)
        if let _ = try? values.decode(Bool.self, forKey: .started) {
            self = .started
            return
        } else if let valStruct = try? values.decode(OneValue&lt;String>.self, forKey: .storedToFile) {
            self = .storedToFile(path: valStruct.value)
            return
        } else if let valStruct = try? values.decode(TwoValues&lt;String,Int>.self, forKey: .uploadingToDevice) {
            self = .uploadingToDevice(fromPath: valStruct.value1, percentage: valStruct.value2)
            return
        } else {
            throw EncodingError.dataCorrupted
        }
    }</code></pre>

It’s all works good but when you develop it don’t rely to much on autocomplete:&nbsp;

Another approach is to use plain key structure like this:

<pre class="wp-block-code"><code>enum UpdateState: Codable {
    case started
    case storedToFile(path: String)
    case uploadingToDevice(fromPath: String, percentage: Int)

    enum CodingKeys: CodingKey {
        case started
        case storedToFilePath
        case uploadingToDevicePath, uploadingToDevicePercentage
    }

    // ...
}</code></pre>

If we have no associated values per case we put some default one. If we have just one value we put this value for the case key (if we have case named **storedToFile** and an associated value **path**, we might call the key **storedToFilePath**). If we have more than 1 associated values we simply create keys for each: **uploadingToDevicePath**, **uploadingToDevicePercentage**.

The encode/decode-methods will look like following:

<pre class="wp-block-code"><code>    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        switch self {
        case .started:
            try container.encode(true, forKey: .started)
        case .storedToFile(let path):
            try container.encode(path, forKey: .storedToFilePath)
        case .uploadingToDevice(let fromPath, let percentage):
            try container.encode(fromPath, forKey: .uploadingToDevicePath)
            try container.encode(percentage, forKey: .uploadingToDevicePercentage)
        }
    }

    init(from decoder: Decoder) throws {
        let values = try decoder.container(keyedBy: CodingKeys.self)
        if let _ = try? values.decode(Bool.self, forKey: .started) {
            self = .started
            return
        } else if let path = try? values.decode(String.self, forKey: .storedToFilePath) {
            self = .storedToFile(path: path)
            return
        } else if let fromPath = try? values.decode(String.self, forKey: .uploadingToDevicePath),
            let percentage = try? values.decode(Int.self, forKey: .uploadingToDevicePercentage) {
            self = .uploadingToDevice(fromPath: fromPath, percentage: percentage)
        } else {
            throw EncodingError.dataCorrupted
        }
    }</code></pre>

This approach a bit more messy because on the same level we have all the keys for case themselves and for the associated values. But the structure is simpler: we don’t have any value wrappers, no general names as well.

Technically the compiler can generate Codable implementation for the enum with associated values as it does it for the rest types, but it’s not clear which structure for the output data to use. There is [a thread](http://(https://forums.swift.org/t/automatic-codable-conformance-for-enums-with-associated-values-that-themselves-conform-to-codable/11499) at swift forum where automatic approach for this case is being discussed. But so far it doesn’t seem we gonna have any solution on a compilation level any time soon. So all is left for us is an implementation of Codable methods.

_(More about native serialization can be found in this post: [Object serialization in iOS](https://dmtopolog.com/object-serialization-in-ios/))_
