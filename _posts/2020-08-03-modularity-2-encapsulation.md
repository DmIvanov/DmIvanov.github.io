---
title: Modularity. Encapsulation
date: 2020-08-03
author: topolog
layout: post
permalink: /modularity-2-encapsulation/
image:
  path: images-posts/2020-08-03-modularity-2-encapsulation/separation-1920.jpg
  thumbnail: images-posts/2020-08-03-modularity-2-encapsulation/separation-600.jpg
  caption: Photo by Will Francis on Unsplash
tags:
  - modules
  - iOS
  - architecture
  - no code
categories:
  - Tech Blog
---

What is this post about:
- what modularity gives us in terms of context separation
- what can help us to keep the module properly encapsulated
- what is a Showcase app and how it can improve our development
- what might be some possible challenges

This post is part of the series on modularity:
- [Modularity. Boundaries](https://dmtopolog.com/modularity-1-boundaries/)
- [**Modularity. Encapsulation**](https://dmtopolog.com/modularity-2-encapsulation/)

&nbsp;

### Module interface.

Module is a complete piece of functionality restrained by the Module Public Interface (MPI). And in the case of a library or a framework, this interface is very formal. No data flows between the module and the host app can bypass MPI. All the module's dependencies are also explicit and can be easily checked. So modularity gives you a better overview of the functionality within the module. You can easier imagine the module's place in the project. You can clearly see what data goes in and out of the module. You can easier estimate the effort to add, remove, or replace it.

On the other side, strict boundaries imply a more tedious procedure to change them. When different parts of a host app (or even several host apps) rely on the interface it shouldn't be easy to modify it. You have to carefully consider if the modification breaks the interface (backward incompatible) or not. If so you have to think about the way to minimize these breaking changes and even provide a migration plan to the module consumers.

Modules require documentation. If you want to have an isolated piece of code (your module) and not feeling the need to check the implementational details every time, proper documentation of the MPI is your choice. It will cost you some additional time to cover the module interface with documentation, but it will pay you back with the amount of time saved. True encapsulation cannot exist without a clear understanding of what the module does, how the consumer supposes to use it, what are the inputs and outputs. And there are basically two ways to get it: jumping into the implementation of the module or checking the documentation. The ladder is much quicker and nicer than the former.

Needless to say that MPI documentation (as every documentation) requires constant maintenance. You might add some dedicated steps in your work process/pipeline in order not to forget about documentation. For instance, you can add a separate checkbox in your module's CI when you release a new module version: `update the documentation if the MPI changes`.


### Context separation.

One of the greatest benefits of modularisation is context separation.

You all know that our brains are not capable of holding big chunks of information for a long time. So when working on the code (writing or debugging) the lesser context you need to keep in mind the better.

![](/images-posts/2020-08-03-modularity-2-encapsulation/IMG_E0532-2000.JPG)

In a badly modularised project, different pieces of functionality are tightly coupled to each other. So when you work on, let's say, some networking code you might need to consider not only this piece of code but several parts of business logic which use this code, all the state which is changed by this code and maybe even a persistency functionality which is wired into this code. Basically, you need to keep in mind half of your app.

If you work on a properly encapsulated networking module on another hand, you care only about the actual networking code and the MPI of the module. The module interface cuts all the functionality into the internal module context and the external one. And in most cases, you don't need to know anything about the external context when working inside the module. So work gets easier, you spend less time on it and it costs you much less cognitive energy to do it.

The ability to treat modules independently from the rest of the app puts you in a better position when you need to refactor or modify the code. You have fewer dependencies (and the ones you have are more clear) and fewer side effects. Consequently, it's also easier to test this code, to mock the dependencies, and to detach the code you test from any global state.


### Showcase app.

If your module is a feature module (it has some UI as well as business logic) you may want to introduce a Showcase app for the module.

Showcase app is a representation of the features and use cases your module provides. It's feasible to make one only if you can separate the feature from the rest of the code both: regarding the UI and regarding the business logic. Otherwise, you will need to include a lot of irrelevant code to the showcase app, and it will be not so different from your major app.

![](/images-posts/2020-08-03-modularity-2-encapsulation/IMG_E0533-2000.JPG)

Let's imagine we have a Profile feature module. It has a UI with the ability to display and modify profile settings. It also contains business logic to connect to a back end to fetch and update the data. So all of this is wrapped into a framework which we use in our major application.

If we create a showcase app for this module it will likely look like a simple app displaying the profile page and maybe several more pages to edit the info. The profile-framework is a dependency of this app. When the app launches we inject some mock network layer to the framework (replacing remote calls with some refilled JSON data) and get the profile screen back from it. So nothing more than our module provides.

First of all, as it follows from the name, `showcase app` tells us what the module has and how its functionality can be used. In the profile showcase app, you might be able to input some user data and see this data on the screen. Under the hood, the module will do all the regular work: sending the data to the (mock) backend and displaying it. The module works exactly the same as in the real app.

This `showcasing` helps to display the capabilities of the module to the product owners and to module consumers. It might be easier to check the changes in the business logic in this simple app then in the main project. You don't need to navigate to the profile all the time, and when using the showcase app you are more focused on the feature.

This app is a simplified host app. You can easily display all the main use cases there as well as some edge cases. For instance, error handling is generally not so reproducible while debugging but it might be important to see how the errors are being handled. You can create a specific mock response from the backend with an error and a separate use case in the UI to trigger this response. So with one tap you can reproduce the error and see how your feature reacts to it.

With such a sample-app you present not only visual functionality to the product owners but also the way to use the MPI to the integrators. Generally, the module interface should be as simple as possible, intuitive, and properly documented. But in reality, all the interfaces can be misunderstood and wrongly used. Hence it's very helpful to display this right approach and good practices in code.

As a developer who needs to integrate a new module in my project I'd always like to check some examples where this module is already integrated: how to set up the configuration, which objects to instantiate, how to handle callbacks. Showcase app is a great way for module maintainers to provide all this information.

It also easier to develop and debug your module utilizing this small focused app. You get faster compilation and waste no time on the navigation to the feature after every app relaunch. Think about how many times you do this routine (change code - recompile - relaunch - navigate to the feature) during the workday. And when relaunching the app is a matter of a couple of seconds you don't switch to checking your facebook feed in between.


### Encapsulation is never perfect

Sometimes there are still connections between 2 properly separated modules. For instance, some changes in one module require changes in another. Or some functionality simply migrates from one screen (in module1) to another (in module2). In this situation, you cannot develop and integrate a new version of the module completely independently. It’s necessary to synchronize development and releases of 2 (but maybe even more) separate modules. When they are maintained by different teams with separate backlogs and priorities, that adds quite a complexity, which inevitably slows down the delivery of the change.

You can work on a perfectly separated module, but there is always a stage when the module (or its new version) needs to be integrated into the app. That’s where the development is not much different compared to a non-modular approach. You are in between the module and the host app so you have to switch from your cozy small module context to the entire app.

Moreover, when integrating the module (or debugging this integration) you might encounter some technical difficulties. For example, it might be challenging to step between the module and the host app code in the debugger because it’s not compiled at the same time. Not talking about the misalignment between the module and the host app. Some cache might not be updated and as a result, you use the old version of the module… which compiles without a problem but causes some weird runtime issues.

We will touch these points in detail again later when talking about the work process peculiarities of developing a modular app.

&nbsp;

---
I hope you liked this piece of reading. If you have any questions, suggestions or corrections you can reach me out [on Twitter](https://twitter.com/dmtopolog)
