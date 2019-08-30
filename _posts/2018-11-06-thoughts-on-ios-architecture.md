---
id: 117
title: Thoughts on iOS Architecture
date: 2018-11-06T09:00:09-02:00
author: topolog
layout: post
guid: https://dmtopolog.com/?p=117
permalink: /thoughts-on-ios-architecture/
image: /wp-content/uploads/2018/11/AppleDoc_threeLayers.png
categories:
  - Tech Blog
tags:
  - architecture
  - iOS
  - patterns
---
Seems like in software development and in iOS particularly yet another post about the architecture looks banal if not just boring. But believe me I won&#8217;t compare MVVM to VIPER or tell you how bad it is to have Massive View Controller. I will not explain you why you should follow a single responsibility principle, why DI is good or what are the trade-offs of using singletons &#8211; I assume you already know that (if not, there are tons of articles about it). This post is more as an attempt to structure my own thoughts on this topic. There will be more questions asked than answers given.

&nbsp;

### Different types of patterns.

Quite often when people talk about patterns they confuse two different flavours of the meaning. Let&#8217;s distinguish **low-level patterns** (the way to solve some problem within one object or between several objects) and **high-level**&nbsp;**patterns** (the way to construct the app, the module, the feature). Low-level patterns like Factory, Builder, Adapter, Decorator and all the rest came through Kent Beck and GoF from the real architecture (where they make buildings.. not the software) and are quite general for all the software development areas. High-level patterns in contrary are platform specific and vary quite a lot between different software realms. If you say &#8220;Singleton&#8221; or &#8220;Facade&#8221; most of the software developers understand what exactly you mean. If you mention VIPER or MVVM more likely just the mobile developers would know what are you talking about. Even worse, if you say &#8220;MVC&#8221; you might be understood differently depending on who are you talking to (which is a bit sad, considering that originally patterns were about common language between all the developers).

High-level patterns may consist of low-level ones and MVC is a good example of it. Here are a quote and an image from Apple Documentation:

> Model-View-Controller is a design pattern that is composed of several more basic design patterns.

<img class=" wp-image-118 aligncenter" src="https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/AppleDoc_MVC_lowLewelPatterns.png?resize=651%2C243&#038;ssl=1" alt="" width="651" height="243" srcset="https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/AppleDoc_MVC_lowLewelPatterns.png?resize=300%2C112&ssl=1 300w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/AppleDoc_MVC_lowLewelPatterns.png?resize=768%2C286&ssl=1 768w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/AppleDoc_MVC_lowLewelPatterns.png?resize=1024%2C382&ssl=1 1024w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/AppleDoc_MVC_lowLewelPatterns.png?resize=1600%2C596&ssl=1 1600w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/AppleDoc_MVC_lowLewelPatterns.png?w=1376&ssl=1 1376w" sizes="(max-width: 651px) 100vw, 651px" data-recalc-dims="1" /> 

If we consider that **pattern** is _a general, reusable solution to a commonly occurring problem within a given context,_ for low-level ones the problem and the context vary from one pattern to another. Some of them are about creating the objects, the others reveal some structural or behavioural peculiarities. But high-level patterns usually solve generally the same problem in different ways: how to design the system. However the context can be slightly different: an application, a service, a module, a screen.

In this post I&#8217;ll be talking about the high-level or **architectural** **patterns**.

&nbsp;

### UI-centric architecture

There are plenty of iOS architectural patterns and new ones appear every year. I really appreciate the joke from Guilherme Rambo: <https://iosarchitecture.top/> It wouldn&#8217;t have been so funny if it hadn&#8217;t had a good portion of truth underneath. I cannot imagine how many new patterns may appear between the moment I write this text and the moment you read it 😉

All the popular architectural models come from MVC (or from other models which came from MVC) and they all share the same concept of screen as the basic architectural unit. iOS app is indeed a number of screens so the pattern should tell you how properly organise them, how to avoid an inflation of a view controller, how to decouple presenting logic from business logic. If you are lucky your favourite pattern might briefly mention something outside the screen (for instance, recommend you to move the routing logic out of the module). But most of them don&#8217;t care much about this &#8220;outside&#8221;, for some of them there are no &#8220;outside&#8221; at all. They are primarily focused on UI screens.

I think we have this situation because originally iOS apps didn&#8217;t have much logic and data manipulation. UI was the paramount and the most important part of the app (and that&#8217;s still the case for lots of apps out there). But nowadays apps become more and more complex. Persistency, networking, data transformation turned from view controller satellites into full-fledged layers and even separate frameworks. Depending on its specifics the app might have extensive work with payments, maps, external devices, huge pieces of logic dedicated to data synchronisation or third party API aggregation. You might need to share some state related to some worker or pass the entire service between different UI parts. Specific services (daemons) might work in the background with minimal to no UI. There are some iOS applications which acquired entire &#8220;core/service layers&#8221; (with dedicated teams maintaining this modules). What place all this modules take in MVVM? Maybe they are mentioned in MVP? What about VIP? VIPER?

Some patterns do mention these responsibilities as _services_, _helpers_ or _workers_, but don&#8217;t pay much attention to them. In the best case it&#8217;s just one block in the scheme among all the Presenters, Interactors and Routers, but this block can mean the entire layer or dozens of object. This layer can be more complex than the screen related logic all together (Presenter + View + Entity + whatever else you have). There are lot&#8217;s of questions about this parts of the code. How do you construct these modules inside? Who should instantiate and own them? How should they be passed between screens and layers? Can they depend on each other? Can one service/worker/helper own another? Is it OK for some of them to be singletons? And so on. And it would be nice to have some system, _some patterns&nbsp;_to be able to answer them.

With moving from MVC to some alternatives we managed to admit that view controller shouldn&#8217;t be the centre and the core of the module, but we still operate screens as major bricks of architecture. Maybe that&#8217;s the time to consider the UI as just one of the app&#8217;s I/O module, an important but not the major driver of it?

&nbsp;

### MVC

> In the beginning was the Word&#8230;

&#8230;actually the acronym, but who cares. In the dawn of iOS there was nothing but what Apple suggested as the way to design our apps.

As we already mentioned all the high-level patterns came from MVC. Every one of them in its description starts with flaws and problems of MVC. And basically all of them consider _Massive View Controller_ as the main (if not the only) issue.

The logical chain of reasoning is:

  1. Apple promotes MVC as the way to design iOS apps.
  2. Every module (screen) should be build with MVC in mind.
  3. UIViewController, representing &#8220;C&#8221; in MVC, incapsulates too many responsibilities.
  4. Too many responsibilities in one object causes lots of negative effects like huge size, tightly coupled code, lack of testability.
  5. To solve this issue we need to split view controller into parts with smaller responsibilities
  6. Depending on what the authors reckon as the major responsibilities they propose several objects instead of one view controller. They call the set of the objects as &#8220;module&#8221;, explain how properly create the parts of the module and what are the rules of interaction between these parts. And vu&#8217;a la an iOS architecture.

All these points generally make sense but as usual the devil is in details. On my opinion there are some inaccuracies in the first couple of statements which makes the rest of the logical chain unreasonable in the current context.

Let&#8217;s start with the first one: \`Apple promotes MVC as the way to design iOS apps\`.

Some developers say that Apple uses MVC in all the sample apps and code examples. But think about it: all this samples are description of some system APIs, usage of some frameworks or just some low-level approaches. All this bites of code are not about high level architecture. It would be too distracting to use proper high-level (or even low-level) patterns when describing something not directly related to it. Imagine if explaining you what is all-season car tires I would like to draw you a picture of the complete car constitution. Would it be what you are interested in? I don&#8217;t think so. So if Apple put all the code into AppDelegate when explaining new CoreBluetooth API let&#8217;s not consider they really mean it, it&#8217;s just simplification.

Talking about the official documentation, MVC indeed is the only high-level pattern mentioned in manuals and guides from Apple. Not just mentioned, but also suggested to base your apps on due to various arguments. There are some detailed explanation what should be considered as view/controller/model and how these parts should communicate to each other. But most of it related to using MVC in Cocoa &#8211; not Cocoa Touch. Of course there are a lot of basic similarities in how macOS/iOS apps function but there are also some significant platform differences which we have to consider.

Then we have to admit that, although the theoretical part of the explanation is quite essential, there is not much official documentation about how to implement it in practice, just some general words. Not saying about the lack of sample projects dedicated to this matter. Apple encourages us to use MVC in our iOS apps but doesn&#8217;t tell us how exactly to do that. As a result all of the thoughts, articles and approaches about MVC applied to iOS are just interpretations, some personal views on the subject. Maybe the problem is not in MVC? Maybe we just don&#8217;t understand how to use the pattern in our apps?

_(I chose the quote from Bible at the beginning of this paragraph because sometimes the guidelines are like the holy book which doesn&#8217;t give you exact answers but needs some interpretation and some prophets.. wait.. we do already have&nbsp;Evangelists)_

&nbsp;

### Different interpretation of MVC

First of all let&#8217;s try to guess how Apple means to use MVC in the real apps. Here are two quotes from the documentation:

> It is a high-level pattern in that it concerns itself with the global architecture of an application and classifies objects according to the general roles they play in an application.
> 
> The MVC design pattern considers there to be three types of objects: model objects, view objects, and controller objects. The MVC pattern defines the roles that these types of objects play in the application and their lines of communication.

For me it seems that MVC should be used in the context of the entire app and not in the context of one screen. Here is the picture from one of the Apple documentation pages:

<img class="wp-image-119 aligncenter" src="https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/11/AppleDoc_threeLayers.png?resize=608%2C363&#038;ssl=1" alt="" width="608" height="363" srcset="https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/11/AppleDoc_threeLayers.png?resize=300%2C179&ssl=1 300w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/11/AppleDoc_threeLayers.png?resize=768%2C458&ssl=1 768w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/11/AppleDoc_threeLayers.png?resize=1024%2C611&ssl=1 1024w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/11/AppleDoc_threeLayers.png?w=1032&ssl=1 1032w" sizes="(max-width: 608px) 100vw, 608px" data-recalc-dims="1" /> 

So the other point of that logical chain &#8211; \`Every module (screen) should be build with MVC in mind\` &#8211; is not so true as well.

You can see that **Model/View/Controller** are layers inside the app, not objects which constitute the screen. So we have to consider this layers at the higher level of abstraction. Controller-layer, with the Application Delegate as one of the _controllers_, symbolises the app&#8217;s business logic. View-layer resides between the business logic and the actual layout. For me it&#8217;s clear that all the view controllers (together with their View Models, Presenters and Interactors) are parts of the view-layer. All this objects basically work with views, some models which supposed to be displayed in the views and process UI events. It doesn&#8217;t matter for this high-level interpretation of MVC what patterns you use for building your screens: MVVM, VIP or VIPER, because it&#8217;s just different implementation of your view layer. You just need to properly separate M, V and C layers and provide the interaction between them according to the MVC-scheme.

Let&#8217;s change the scheme just a bit so it looks like that:

<img class="alignnone wp-image-150" src="https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/DiffernetMVC.png?resize=688%2C252&#038;ssl=1" alt="" width="688" height="252" srcset="https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/DiffernetMVC.png?resize=300%2C110&ssl=1 300w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/DiffernetMVC.png?resize=768%2C283&ssl=1 768w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/DiffernetMVC.png?resize=1024%2C377&ssl=1 1024w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/DiffernetMVC.png?resize=1600%2C589&ssl=1 1600w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/DiffernetMVC.png?w=1376&ssl=1 1376w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/DiffernetMVC.png?w=2064&ssl=1 2064w" sizes="(max-width: 688px) 100vw, 688px" data-recalc-dims="1" /> 

If you look at MVC from this point of view you can see how it aligns with Uncle Bob&#8217;s architectural layers. So you can apply most of the Clean Architecture principles here:

  * The Dependency rule (higher layer depends on the lower layer, lower layer know noting about higher layer)
  * Clear borders and API between the layers. The layer interacts only with his _neighbour-layers_.
  * DTO (Data Transfer Objects) should be used to pass data between layers
  * The changes in higher layer shouldn&#8217;t influence the lower layer. Changes in the lower layer may influence the higher layer.

Basically that are the same principles which mentioned in Apple&#8217;s description of MVC rephrased a bit differently. Maybe Clean Architecture and MVC are not so far from each other?

Let&#8217;s come back for a moment to the chain of reasoning which and its point \`UIViewController represents &#8220;C&#8221; in MVC\`. Having all the ideas above, the point can be seen from several different perspectives.

One can say now that&nbsp;UIViewController has nothing to do with MVC (not everything that is called _model_, _view_&nbsp;or _controller_&nbsp;is part of MVC pattern).UIViewController is just one of the view-layer objects with its responsibilities (mostly &#8211; managing the views) and connections with the other objects. These other objects are not limited by _model_ and _view_. So you should forget about MVC on this level and treat it with the general approach (SOLID, DRY, KISS and all the rest old but useful stuff).

The other one can say, we still can consider the screen as an MVC module.UIViewController as an&nbsp;_objects with merged MVC roles_ (the term from Apple documentation) communicates with the model, BUT the model here is some data to display on screen, nothing more than that. This data comes through some Interactors, Workers and Transformers in the business logic of the app which are outside of this very UI-related MVC module. And when all the persistency, networking, third party libraries and devices are out of our MVC module there are not so much work left to do, so view controller can easily communicate to it&#8217;s model and layout it&#8217;s views without embracing too much things to care about.

Both these approaches to view controllers give you enough freedom to write properly structured dumb view controllers without much logic so you can keep it small and clean. But the idea of higher level MVC is the same.

### 

### Layers constitution

So as we can see, **MVC is broken**-complaint usually comes when the speaker consider MVC only in the context of the separate screen when UIViewController incapsulates not only view logic but also a decent part of business logic. Let&#8217;s imagine I succeeded in persuading you that it should be treated differently. Ok, we have UI, Business logic and core layers, what&#8217;s next?

If your application is big enough (otherwise you happily use some MVVM and has already stopped reading this) you will notice that your layers are huge. You might have hundreds and thousands classes which are considered as parts of one layer and that&#8217;s totally ok. It doesn&#8217;t mean your presenting or business layer should be a big bowl of spaghetti, it doesn&#8217;t mean the classes inside it should be tightly coupled to one another and violate the rest of the basic OOP principles. No. Just zoom in! Go to a bit deeper level of abstraction.

<img class="wp-image-120 aligncenter" src="https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/Layers_zoomIn.png?resize=383%2C497&#038;ssl=1" alt="" width="383" height="497" srcset="https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/Layers_zoomIn.png?resize=231%2C300&ssl=1 231w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/Layers_zoomIn.png?resize=768%2C998&ssl=1 768w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/Layers_zoomIn.png?resize=788%2C1024&ssl=1 788w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/Layers_zoomIn.png?resize=1600%2C2079&ssl=1 1600w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/11/Layers_zoomIn.png?w=1376&ssl=1 1376w" sizes="(max-width: 383px) 100vw, 383px" data-recalc-dims="1" /> 

Of course it&#8217;s better to design some clear contracts and strict APIs between the layers. Don&#8217;t forget about Clean Architecture principles. But other than that you can consider your layers as kind of a separate systems.

Because of the UI-centric approach to the architecture we have plenty of options for the UI layer architecture, but not so much guidelines about our business logic. With core layer there are less questions as core components suppose to be incapsulated independent modules with their own APIs.

<img class="wp-image-121 aligncenter" src="https://i2.wp.com/dmtopolog.com/wp-content/uploads/2018/11/LayersAndCorrespondingPatterns.png?resize=489%2C189&#038;ssl=1" alt="" width="489" height="189" srcset="https://i2.wp.com/dmtopolog.com/wp-content/uploads/2018/11/LayersAndCorrespondingPatterns.png?resize=300%2C116&ssl=1 300w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2018/11/LayersAndCorrespondingPatterns.png?resize=768%2C297&ssl=1 768w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2018/11/LayersAndCorrespondingPatterns.png?resize=1024%2C396&ssl=1 1024w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2018/11/LayersAndCorrespondingPatterns.png?resize=1600%2C619&ssl=1 1600w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2018/11/LayersAndCorrespondingPatterns.png?w=1376&ssl=1 1376w, https://i2.wp.com/dmtopolog.com/wp-content/uploads/2018/11/LayersAndCorrespondingPatterns.png?w=2064&ssl=1 2064w" sizes="(max-width: 489px) 100vw, 489px" data-recalc-dims="1" /> 

Regarding business logic architecture in iOS I&#8217;m aware of just one approach &#8211; SOA ([Service Oriented Architecture](https://en.wikipedia.org/wiki/Service-oriented_architecture)).&nbsp; There are some variations of it for iOS, but basically it says that your business logic is the set of services responsible for specific types of models and operations under these models. Services should be UI independent so it will be easier to reuse them. Then there are some variations about how to deal with the state in the services and whether they can depends from one another. But briefly that&#8217;s the idea.

To be fair I can partly understand why the discussion about this layer is less popular. First of all, I&#8217;ve already mentioned that there are plenty of apps which don&#8217;t have so much of business logic, so picking the proper UI pattern is enough for them to be in a good shape. The second reason might be that business logic is much more different from app to app than UI. Although there are lots of room for UI-creativity, in terms of implementation we all are restricted to use UIViewControllers and UIViews. Hence the challenges in presentation layer are quite similar for everybody. Regarding business logic there are less platform restrictions and more app-specific variety, so it&#8217;s really hard to talk about some general approaches. But it shouldn&#8217;t prevent us from thinking about patterns for this deeper layers. For each set of challenges there are more and less appropriate solutions, there are right and wrong architectural decisions. Maybe the story of one application will be not 100% relevant to another app, but there definitely are some common problem and challenges.

I hope it can encourage you to start thinking of you business logic layer from this angle (if you haven&#8217;t done it yet) and share your approach (if you already have any). I think it might be an interesting piece of reading/listening 😉

&nbsp;

### Several UI patterns in one app

Let me take the last couple of minutes of your time and come back to the UI patterns. One of the question about them which has always been interesting for me is: &#8220;Does it make sense to use different UI patterns in one app&#8221;?

Let&#8217;s say we use some variation of VIP (Clean Swift) in our application, but for some module we would like to try VIPER or MVVM + some\_reactive\_framework. Would it be confusing? What about some simple module where UI version of MVC is more that enough? Imagine some primitive view controller with an image, couple of labels and one button. If we sure we are not going to extend it, should we really build all the Presenters-Interactors together with covering protocols just in sake of consistency?

On one hand patterns are our language, the way we communicate ideas among each other. If we have several words in our disposal why should we use just one? If MVVM is more suitable _word_ to describe the idea of the screen, why should we use verbose VIBER or primitive MVC for it? We have screens of different complexity inside the app, so if we agreed to use just one pattern, for some screens it might be an overhead, but the others might require even more complex solution (like splitting some Presenter or Interactor in parts due to increasing amount of responsibilities).

On the other hand using several patterns which has the same naming convention might be really confusing. &#8220;Interactor&#8221; in VIPER and in VIP is not the same, so to get its area of responsibility you have to understand the construction of the module first. If you are looking for the object responsible for presenting view controller on screen your search query depends on the pattern. When you start implementing the screen you might pick the easiest pattern which fit your requirements, but the requirements may change later and you might need to choose the other pattern and refactor the screen according it.

For me it seems just easier to stick to some pattern (not the most complex one) and just slightly refactor it if necessary without changing the roles of major actors (create some dependent objects, split responsibilities). But I cannot say I&#8217;m sure that&#8217;s the best way to handle it.

&nbsp;

Got any ideas or objections? Don&#8217;t hesitate to drop me a line in comments or contact me any other way. I&#8217;d like to know what do you think about all that.

&nbsp;

**_Useful or interesting references on the matter:_**  
_Apple&#8217;s MVC:_  
_<https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/Model-View-Controller/Model-View-Controller.html>_  
_<https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/Introduction/Introduction.html>_

_Model-View-Controller (MVC) in iOS: A Modern Approach:_  
_<https://www.raywenderlich.com/1073-model-view-controller-mvc-in-ios-a-modern-approach>_

_Dave DeLong&#8217;s view on MVC (the first post from the series and corresponding talk):_  
_<https://davedelong.com/blog/2017/11/06/a-better-mvc-part-1-the-problems/>_  
[_https://youtu.be/YWVzCd5FYbs_](https://youtu.be/YWVzCd5FYbs)

_Krzysztof Zabłocki &#8211; iOS Application Architecture (partly covering similar ideas)_  
[_https://youtu.be/PdkWjdKOqfo_](https://youtu.be/PdkWjdKOqfo)

_Clean Architecture:_  
_<http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html>_

_(Russian) Стас Цыганов &#8211; Сервис-ориентированная архитектура_  
[_https://youtu.be/Eman1j06YsU_](https://youtu.be/Eman1j06YsU)