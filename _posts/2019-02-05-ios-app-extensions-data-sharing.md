---
id: 260
title: 'iOS App Extensions: Data Sharing'
date: 2019-02-05T07:19:26-02:00
author: topolog
layout: post
guid: https://dmtopolog.com/?p=260
permalink: /ios-app-extensions-data-sharing/
image:
  path: images-posts/2019-02-05-ios-app-extensions-data-sharing/CordExtension-2000.jpg
  thumbnail: images-posts/2019-02-05-ios-app-extensions-data-sharing/CordExtension-600.jpg
categories:
  - Tech Blog
tags:
  - AppExtension
  - DataSharing
  - iOS
---

App extensions are permeating more and more into our daily job. But some nuances of the way they work are still not so obvious. In this post I try to investigate different ways to share data between an app and its extension.

> An app extension lets you extend custom functionality and content beyond your app and make it available to users while they’re interacting with other apps or the system.
>
> -[_Apple documentation_](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/index.html#//apple_ref/doc/uid/TP40014214-CH20-SW1)

An extension concept lets your code to run outside the actual iOS application with or without some dedicated UI. To create an extension you pick an appropriate target template, adjust the configuration, assign files and resources to be build and eventually you have a separate binary file. This separate product has its own sandbox and runs independently.

There are plenty types of iOS app extensions: Custom Keyboard, iMessage, Share, Today Widget, WatchKit App, Intents and several more.

&nbsp;

## Containing App vs. Host App

Talking about app extension (or reading Apple’s documentation about them) it’s important to understand what is “host app” and “containing app”.

**Containing app** is the app you extend with an extension, the main app you are developing. It might or might not have any shared code or resources with your extension, but they are bundled and installed to the device all together. The primary function of the containing app is to deliver an extension to the user. When your app is installed on a device and your extension is registered in the system (which is done automatically by iOS and might take several minutes) your extension is delivered. Theoretically after that an extension may have no connections to its containing app.

**Host app** is the app which runs your extension, in which context the extension works. The entire extension API works in a request-response way: extension is being activated by some request, it does some job and returns some response. After that the system kills it. The host app is the one who exchanges these requests and responses with your extension.

![](/images-posts/2019-02-05-ios-app-extensions-data-sharing/containing-app-host-app.png)

Third party apps, system apps or even some parts of iOS itself may be the “host app”. If you use Share Extension or Custom Keybord Extension every app which has share-button or input text field can host it. System Today View hosts all the Today widgets. In case of Intents Extension Siri plays the host role and communicate with your code.

&nbsp;

## Data sharing

As I mentioned an extension is not suppose to interact much with the containing app. In some case like Watch Extension there are some dedicated API for this communication. But typically you don’t have this channels. So thinking about the functionality you want your extension to implement consider this restriction.

It’s hard to imagine that your extension will work totally independent from your main app. More likely you’ll need to share some data or resources. Talking about Intents Extension there are several ways how it can be done.

#### **1. Remote API**

With this approach you don’t share anything between the app and the extension locally on a device. Your extension might just get whatever is needed from the internet itself (from your own remote API or from some third party). In this case you have to keep in mind networking you’ll have to use in your extension (more about this later on).

Beware that in Intent Extension you are restricted in time your code can perform an operation. Is you don’t return a response to Siri within 5-10 second it will not wait longer and behave like some error occurred. That might be crucial when you do network call from the extension.

#### **2. User Defaults Group**

That’s the easiest and the most common approach which will suit you in the majority of cases. You have to enable app groups in both your app’s and extension’s targets as well as at the developer’s portal. Then you are able to use UserDefaults container which can be shared between several apps (or between the app and the extension).

Let's for instanse wrap it into UserDefaults extension:

```swift
extension UserDefaults {
  static let group = UserDefaults(suiteName: "group.com.your.domain")!
}
```

setting data:
```swift
UserDefaults.group.set("objectToShare", forKey: "keyForTheObject")

```

getting data:
```swift
let object = UserDefaults.group.object(forKey: "keyForTheObject")
```

_(Beware that for some reason this approach doesn’t properly work on iPhone Simulator)_

#### **3. Shared container**

After enabling app groups you are also able to use shared container - the entire folder in the file system. We can wrap it into an extension as well:

```swift
extension FileManager {
  static func sharedContainerURL() -> URL {
    return FileManager.default.containerURL(
      forSecurityApplicationGroupIdentifier: "group.com.your.domain"
    )!
  }
}
```

You can use this URL to read/write whatever is necessary. Just remember that access to the container is not thread safe so you might need to use some locks to prevent data corruption. Starting from iOS 9 we are able to use [NSFileCoordinator](https://developer.apple.com/documentation/foundation/nsfilecoordinator) class designed specifically for that. It provides thread safe access to the file system from different processes (the class exists from iOS 5, but it was modified for using with shared container). Some more details about using NSFileCoordinator and NSFilePresenter in [this post](https://www.atomicbird.com/blog/sharing-with-app-extensions)

#### **4. Data Base sharing.**

It’s a specific case of the previous type. If you need to sync some more or less complicated object graphs which you store in some kind of data base (SQLite, CoreData, Realm) in your main app. You can share the access to the data base source file in your file system using shared container and recreate the data base within your extension. In case of SQLite (or Core Data backed by SQLite) you shouldn’t even worry about thread safety - SQLite is good in smart enough to do it for you.

Here is an example of using shared container to instantiate NSManagedContext backed by shared SQLite storage:

```swift
extension NSManagedObjectContext {
    
    static func mainContextForSharedStorage() -> NSManagedObjectContext {
        let context = NSManagedObjectContext(
          concurrencyType: .mainQueueConcurrencyType
        )
        context.persistentStoreCoordinator = persistentStoreCoordinator()
        return context
    }
    
    static func persistentStoreCoordinator() -> NSPersistentStoreCoordinator? {
        let conteinerURL = FileManager.sharedContainerURL()
        let storeFileURL = conteinerURL.appendingPathComponent("myFileNmae.sqlite")
        let persistentStoreCoordinator = NSPersistentStoreCoordinator(
          managedObjectModel: managedObjectModel
        )
        do {
            try persistentStoreCoordinator.addPersistentStore(
              ofType: NSSQLiteStoreType,
              configurationName: nil,
              at: storeFileURL,
              options: nil
            )
        } catch {
            fatalError("Unable to Load Persistent Store")
        }
        return persistentStoreCoordinator
    }
}

```

#### **5. NSURLSession cache.**

That’s another variation of shared container approach useful if your extension uses remote API (see p.1) - you can assign shared container to NSURLSession. As far as we've started to wrap everything into extensions let's be consistent:


```swift
extension URLSession {
  static func sharedSession() -> URLSession {
    let configuration = URLSessionConfiguration.background(
      withIdentifier: "com.your.domain.someIdentifier"
    )
    configuration.sharedContainerIdentifier = "group.com.your.domain"
    return URLSession(configuration: configuration)
  }
}
```

First of all it lets you to share downloaded data between the app and the extension. It may be useful in both direction (app -> extension or extension -> app) depending on your use case.

Secondly, in the containing app you can finish a session started from the extension. In iOS, if your extension isn’t running when a background task completes, the system launches your containing app in the background and calls the[application:handleEventsForBackgroundURLSession:completionHandler:](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622941-application)app delegate method.

#### 6. **Keychain Group.**

You can share Keychain access more or less the same way as you share User Defaults. It might be useful in case of sharing sensitive data. For example if your extension needs to extensively use your remote API (see p.1) you might decide to share user credentials and/or user token, so the extension doesn’t need to implement authentication - which can be quite challenging for Intent Extension deprived of UI. If you implement this sharing yourself here are [some more details](http://swiftandpainless.com/ios8-share-extension-with-a-shared-keychain/) about it. But if you use some wrapper for Keychain access quite likely this functionality is already implemented there.

#### **7. iCloud sync.**

You can also access shared iCloud if you are brave enough 😉 You can check out Lister sample app published by Apple (for some reason the app was removed from the Apple samples directory and now you can only find it in [some mirrors on GitHub](https://github.com/RommelTJ/Lister))

_More about data sharing in [_official documentation_](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionScenarios.html#//apple_ref/doc/uid/TP40014214-CH21-SW1﻿) and [WWDC 2015 App Extension Best Practices video](https://developer.apple.com/videos/play/wwdc2015/224/)
