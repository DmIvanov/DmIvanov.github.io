---
id: 260
title: 'iOS App Extensions: Data Sharing'
date: 2019-02-05T07:19:26-02:00
author: topolog
layout: post
guid: https://dmtopolog.com/?p=260
permalink: /ios-app-extensions-data-sharing/
image:
  path: images-posts/2019-02-05-ios-app-extensions-data-sharing/CordExtension-1024x665.jpg
  thumbnail: images-posts/2019-02-05-ios-app-extensions-data-sharing/CordExtension-768x499.jpg
categories:
  - Tech Blog
tags:
  - AppExtension
  - DataSharing
  - iOS
---
App extensions are permeating more and more into our daily job. But some nuances of the way they work are still not so obvious. In this post I try to investigate different ways to share data between an app and its extension.

<blockquote class="wp-block-quote">
  <p>
    <em>An app extension lets you extend custom functionality and content beyond your app and make it available to users while they’re interacting with other apps or the system.</em>
  </p>

  <cite><a href="https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/index.html#//apple_ref/doc/uid/TP40014214-CH20-SW1">https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/index.html#//apple_ref/doc/uid/TP40014214-CH20-SW1</a><br /></cite>
</blockquote>

An extension concept lets your code to run outside the actual iOS application with or without some dedicated UI. To create an extension you pick an appropriate target template, adjust the configuration, assign files and resources to be build and eventually you have a separate binary file. This separate product has its own sandbox and runs independently.

There are plenty types of iOS app extensions: Custom Keyboard, iMessage,&nbsp; Share, Today Widget, WatchKit App, Intents and several more.

## Containing App vs. Host App

Talking about app extension (or reading Apple’s documentation about them) it’s important to understand what is “host app” and “containing app”.

**Containing app** is the app you extend with an extension, the main app you are developing. It might or might not have any shared code or resources with your extension, but they are bundled and installed to the device all together. The primary function of the containing app is to deliver an extension to the user. When your app is installed on a device and your extension is registered in the system (which is done automatically by iOS and might take several minutes) your extension is delivered. Theoretically after that an extension may have no connections to its containing app.

**Host app** is the app which runs your extension, in which context the extension works. The entire extension API works in a request-response way: extension is being activated by some request, it does some job and returns some response. After that the system kills it. The host app is the one who exchanges these requests and responses with your extension.

<div class="wp-block-image">
  <figure class="aligncenter"><img src="https://i1.wp.com/dmtopolog.com/wp-content/uploads/2019/02/AppExtension-containing-app-host-app.png?fit=688%2C335&ssl=1" alt="" class="wp-image-243" srcset="https://i1.wp.com/dmtopolog.com/wp-content/uploads/2019/02/AppExtension-containing-app-host-app.png?w=1356&ssl=1 1356w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2019/02/AppExtension-containing-app-host-app.png?resize=300%2C146&ssl=1 300w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2019/02/AppExtension-containing-app-host-app.png?resize=768%2C374&ssl=1 768w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2019/02/AppExtension-containing-app-host-app.png?resize=1024%2C498&ssl=1 1024w" sizes="(max-width: 688px) 100vw, 688px" /><figcaption>from: <a href="https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionOverview.html#//apple_ref/doc/uid/TP40014214-CH2-SW2">https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionOverview.html#//apple_ref/doc/uid/TP40014214-CH2-SW2</a></figcaption></figure>
</div>

Third party apps, system apps or even some parts of iOS itself may be the “host app”. If you use Share Extension or Custom Keybord Extension every app which has share-button or input text field can host it. System Today View hosts all the Today widgets. In case of Intents Extension Siri plays the host role and communicate with your code.

<div style="height:100px" aria-hidden="true" class="wp-block-spacer">
</div>

## Data sharing

As I mentioned an extension is not suppose to interact much with the containing app. In some case like Watch Extension there are some dedicated API for this communication. But typically you don’t have this channels. So thinking about the functionality you want your extension to implement consider this restriction.

It’s hard to imagine that your extension will work totally independent from your main app. More likely you’ll need to share some data or resources. Talking about Intents Extension there are several ways how it can be done.

#### **1. Remote API**

&nbsp;With this approach you don’t share anything between the app and the extension locally on a device. Your extension might just get whatever is needed from the internet itself (from your own remote API or from some third party). In this case you have to keep in mind networking you’ll have to use in your extension (more about this later on).

Beware that in Intent Extension you are restricted in time your code can perform an operation. Is you don’t return a response to Siri within 5-10 second it will not wait longer and behave like some error occurred. That might be crucial when you do network call from the extension.

#### **2. User Defaults Group**

That’s the easiest and the most common approach which will suit you in the majority of cases. You have to enable app groups in both your app’s and extension’s targets as well as at the developer’s portal. Then you are able to use UserDefaults container which can be shared between several apps (or between the app and the extension).

<div class="wp-block-image">
  <figure class="aligncenter"><img src="https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-User-Defaults-Group.png?fit=688%2C128&ssl=1" alt="" class="wp-image-246" srcset="https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-User-Defaults-Group.png?w=1338&ssl=1 1338w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-User-Defaults-Group.png?resize=300%2C56&ssl=1 300w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-User-Defaults-Group.png?resize=768%2C143&ssl=1 768w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-User-Defaults-Group.png?resize=1024%2C191&ssl=1 1024w" sizes="(max-width: 688px) 100vw, 688px" /><figcaption>You can wrap it into UserDefaults extension</figcaption></figure>
</div>

<div class="wp-block-image">
  <figure class="aligncenter"><img src="https://i1.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-User-Defaults-Group-set.png?fit=688%2C99&ssl=1" alt="" class="wp-image-247" srcset="https://i1.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-User-Defaults-Group-set.png?w=1236&ssl=1 1236w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-User-Defaults-Group-set.png?resize=300%2C43&ssl=1 300w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-User-Defaults-Group-set.png?resize=768%2C111&ssl=1 768w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-User-Defaults-Group-set.png?resize=1024%2C147&ssl=1 1024w" sizes="(max-width: 688px) 100vw, 688px" /><figcaption>setting data<br /></figcaption></figure>
</div>

<div class="wp-block-image">
  <figure class="aligncenter"><img src="https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-User-Defaults-Group-get.png?fit=688%2C100&ssl=1" alt="" class="wp-image-248" srcset="https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-User-Defaults-Group-get.png?w=1220&ssl=1 1220w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-User-Defaults-Group-get.png?resize=300%2C44&ssl=1 300w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-User-Defaults-Group-get.png?resize=768%2C112&ssl=1 768w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-User-Defaults-Group-get.png?resize=1024%2C149&ssl=1 1024w" sizes="(max-width: 688px) 100vw, 688px" /><figcaption>getting data</figcaption></figure>
</div>

_(Beware that for some reason this approach doesn’t properly work on iPhone Simulator)_

#### **3. Shared container**

After enabling app groups you are also able to user shared container &#8211; the entire folder in the file system:

<div class="wp-block-image">
  <figure class="aligncenter"><img src="https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-Shared-container.png?resize=688%2C198&#038;ssl=1" alt="" class="wp-image-253" srcset="https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-Shared-container.png?resize=1024%2C294&ssl=1 1024w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-Shared-container.png?resize=300%2C86&ssl=1 300w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-Shared-container.png?resize=768%2C221&ssl=1 768w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-Shared-container.png?w=1372&ssl=1 1372w" sizes="(max-width: 688px) 100vw, 688px" data-recalc-dims="1" /><figcaption>We can wrap it into an extension as well</figcaption></figure>
</div>

You can use this URL to read/write whatever is necessary. Just remember that access to the container is not thread safe so you might need to use some locks to prevent data corruption. Starting from iOS 9 we are able to use &nbsp;[NSFileCoordinator](https://developer.apple.com/documentation/foundation/nsfilecoordinator) class designed specifically for that. It provides thread safe access to the file system from different processes (the class exists from iOS 5, but it was modified for using with shared container). Some more details about using NSFileCoordinator and NSFilePresenter in [this post](https://www.atomicbird.com/blog/sharing-with-app-extensions)

#### **4. Data Base sharing.**

It’s a specific case of the previous type. If you need to sync some more or less complicated object graphs which you store in some kind of data base (SQLite, CoreData, Realm) in your main app. You can share the access to the data base source file in your file system using shared container and recreate the data base within your extension. In case of SQLite (or Core Data backed by SQLite) you shouldn’t even worry about thread safety &#8211; SQLite is good in smart enough to do it for you.

Here is an example of using shared container to instantiate NSManagedContext backed by shared SQLite storage:

<div class="wp-block-image">
  <figure class="aligncenter"><img src="https://i1.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-CoreData-shared-context.png?fit=688%2C369&ssl=1" alt="" class="wp-image-261" srcset="https://i1.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-CoreData-shared-context.png?w=1608&ssl=1 1608w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-CoreData-shared-context.png?resize=300%2C161&ssl=1 300w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-CoreData-shared-context.png?resize=768%2C412&ssl=1 768w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-CoreData-shared-context.png?resize=1024%2C549&ssl=1 1024w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-CoreData-shared-context.png?resize=1600%2C858&ssl=1 1600w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-CoreData-shared-context.png?w=1376&ssl=1 1376w" sizes="(max-width: 688px) 100vw, 688px" /></figure>
</div>

#### **5. NSURLSession cache.**

That’s another variation of shared container approach useful if your extension uses remote API (see p.1) &#8211; you can assign shared container to NSURLSession.&nbsp;

<div class="wp-block-image">
  <figure class="aligncenter"><img src="https://i0.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-URLSession-shared.png?resize=688%2C237&#038;ssl=1" alt="" class="wp-image-252" srcset="https://i0.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-URLSession-shared.png?resize=1024%2C352&ssl=1 1024w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-URLSession-shared.png?resize=300%2C103&ssl=1 300w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-URLSession-shared.png?resize=768%2C264&ssl=1 768w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2019/02/Code-URLSession-shared.png?w=1356&ssl=1 1356w" sizes="(max-width: 688px) 100vw, 688px" data-recalc-dims="1" /><figcaption>&#8230;as far as we&#8217;ve started to wrap everything into extensions</figcaption></figure>
</div>

First of all it lets you to share downloaded data between the app and the extension. It may be useful in both direction (app -> extension or extension -> app) depending on your use case.&nbsp;

Secondly, in the containing app you can finish a session started from the extension. In iOS, if your extension isn’t running when a background task completes, the system launches your containing app in the background and calls the&nbsp;[application:handleEventsForBackgroundURLSession:completionHandler:](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622941-application)&nbsp;app delegate method.

#### 6. **Keychain Group.**

You can share Keychain access more or less the same way as you share User Defaults. It might be useful in case of sharing sensitive data. For example if your extension needs to extensively use your remote API (see p.1) you might decide to share user credentials and/or user token, so the extension doesn’t need to implement authentication &#8211; which can be quite challenging for Intent Extension deprived of UI. If you implement this sharing yourself here are [some more details](http://swiftandpainless.com/ios8-share-extension-with-a-shared-keychain/) about it. But if you use some wrapper for Keychain access quite likely this functionality is already implemented there.

#### **7. iCloud sync.**

You can also access shared iCloud if you are brave enough 😉 You can check out Lister sample app published by Apple (for some reason the app war removed from the Apple samples directory and now you can only find it in [some mirrors on GitHub](https://github.com/RommelTJ/Lister))

_More about data sharing in the_ [_official documentation_](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionScenarios.html#//apple_ref/doc/uid/TP40014214-CH21-SW1﻿) and [WWDC 2015 App Extension Best Practices video](https://developer.apple.com/videos/play/wwdc2015/224/)
