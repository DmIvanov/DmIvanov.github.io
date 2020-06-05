---
title: Modularity. Boundaries
date: 2020-06-08
author: topolog
layout: post
permalink: /modularity-1-boundaries/
image:
  path: images-posts/2020-06-08-modularity-1-boundaries/fence-2000.jpg
  thumbnail: images-posts/2020-06-08-modularity-1-boundaries/fence-600.jpg
  caption: Photo by Krzysztof Walczak on Unsplash
tags:
  - modules
  - iOS
  - swift
categories:
  - Tech Blog
---

In this post we'll talk about why boundaries matter and what types of modularity can we distinguish in our projects based on the existing boundaries.

## Boundaries and interfaces.

When working with code we all the time are dealing with boundaries between different subsystems. It happens on different levels of abstraction. Functions, classes, modules, frameworks, layers are all encapsulated entities separated from the other world by different kinds of boundaries.

An interface is a legal way to cross the boundary. It describes all the possibilities to pass data or execution control between entities.

Boundaries can be physical or purely architectural.

You cannot call the method of a class from another class if it's not public, the same works with frameworks. We have a clear separation of contexts and physical boundaries between different subsystems. The interfaces here are enforced by the compiler, linker, or some other tool. You cannot cross such a boundary without changes in the interface.

But sometimes the boundaries are just marked architecturally. We may separate our classes into architectural layers and set some rules on how these layers should communicate to each other, but there is no tool to enforce the interface.

## Modularity

`Module` is quite a general term. It may mean both a physical separation (compiled library or framework, some package, subproject, different repository) or just an architectural concept without any formal interface. So for clarity let's call them **physical modules** and **architectural module**. The former obviously have some physical boundaries, the latter - don't.

### Architectural modularity

Let's imagine in our Swift project we have UI, business logic and network layers. Within the business logic layer, we have some feature modules. Inside the team, you have some agreements and guidelines like calling the network layer only from the business logic and not from the UI, or not calling one feature from another. All the classes of the modules and layers are properly documented and carefully separated into different folders. But all the files reside in one project and belong to the same target (we will talk about the targets in a minute). It's an example of pure architectural modularity.

![](/images-posts/2020-06-08-modularity-1-boundaries/IMG_E9634.JPG)

The thing is: there are no technical ways to enforce these agreements. Everything may even work and you manage to keep your boundaries and interfaces clean. But it's so easy to break this balance. Nothing stops the developer to do something wrong: for instance, calling NetworkManager directly from a UITableViewCell. Or using some module's "internal" classes from outside of the module.

Why would one do that? A lot of possibilities:
  - you haven't written the rules down
  - it's so easy to forget or confuse something
  - somebody just disagrees with the architectural concepts in use, or maybe mostly agrees, but treats some case as an exception or things that the agreements are outdated
  - a new developer, who doesn't know or doesn't respect the agreements
  - making a shortcut under the pressure of a deadline

As the team of developers working on a product evolves (some people leave, new people come) all these agreements and assumptions stop work at some point. If some shortcuts can be physically done without extra effort, maybe even unintentionally, it will be done sooner or later. So the boundaries will be crossed and interfaces will be bypassed.

So we need some physical boundaries and technical limitations. There should be strict rules and ways to enforce people to follow the rules, to make it more difficult to make mistakes.

### Physical modularity on iOS

When talking about architectural modularity we can abstract from the language as the concepts are common for all the software development. But if we dive into the implementational details we cannot generalise much anymore.

Regarding physical modularity on iOS, we have to distinguish (Objective-)C modules from Swift modules. Modules are being created by the compiler and we have two different compiler frontends - clang and swiftc - which deal with modules differently.

_(More about these different compilation pipelines you can find in one of my previous posts - (Compiler code optimization for Swift and Objective-C)[https://dmtopolog.com/code-optimization-for-swift-and-objective-c/]. If you need more technical insights into clang modularity check out the (documentation)[https://clang.llvm.org/docs/Modules.html] for more info regarding Swift take a look (here)[https://forums.swift.org/t/explicit-module-builds-the-new-swift-driver-and-swiftpm/36990], (here)[https://docs.swift.org/swift-book/LanguageGuide/AccessControl.html] and (here)[https://swift.org/package-manager/])_

Although there are some significant differences between Swift and (Objective-)C modules (name spacing, access control,..) the essence is the same.

> A module is a single unit of code distribution — a framework or application that is built and shipped as a single unit and that can be imported by another module with `import` keyword.

When you split your code into physical modules you create several different units instead of one. Your code can still sit all in one place (repo/workspace/project) but different parts of it are being assembled independently. When one module refers to another they consider each other as different projects with physical boundaries which can be crossed only via some allowed ways - MPI (Module Public Interface).

### Targets and access control

In iOS projects modules are configured by **targets**. Target is a final product that will be compiled out of your source files. It may be a framework or a library, an application, an application extension, or something else. Some of these products can be used as modules.

It's quite intuitive that physical separation of units reinforces the boundaries between them.

If the file belongs to `Target A` and doesn't belong to `Target B` you need to do some deliberate actions to make this file accessible inside `target B`. It's a two-step process. Firstly you need to make the file (or it's part) public, so available from the outside of the module. Secondly, you need to import module `A` into the module `B`.

Access control levels are another tool for boundary reinforcement.

In ObjC (as inherited from C) we have private implementation (.m) and public interface (.h). Hence we have 2 ways of establishing some physical code access boundaries: putting something into a class interface and importing such interfaces in other files. If some functionality of `Class A` is public it's not enough to use it in `Class B`. You also need to explicitly import the interface of `Class A` in `Class B`. That stays the same regardless of whether or not `Class A` and `Class B` are in the same module or in different ones.

In Swift, we have more different access levels (with corresponding modifiers): object-level (`private`), file-level (`fileprivate`), module-level (`internal`), project level (`public`, `open`). Modifiers help us to build clear MPI for the module. Until `Class A` is declared as `public` or `open` nobody can use it outside of its module. So changing the module interface has to be explicit and cannot be made by mistake. It's also something that can be easily spotted during the code review.

As we can see in Swift we are not able to build custom boundaries between architectural modules as we can do it in ObjC by (not) importing interfaces. But we have more options when it comes to modules. In Swift physical modularity does really help you with the boundaries.

### Other boundary enforcements

All your physical modules can be put into separate projects and even separate repositories. In case of including just a binary, you detach two modules even more. The main project in this case has no references to the source code of the dependency.

The next level of separation is using versioning for your modules. In this case, to make a change in the MPI you need to make a change in the module's project, release a new version of the module, and then integrate the new version into your host project. (I'm not saying you need module versioning for making the boundaries stronger. The other way around: if you do have the versioning in place - or planning to have it due to other reasons - it will also enhance the boundaries.)

Different modules can be developed by different teams/departments. This makes all the small changes in the module just to fit the needs of the host app borderline impossible. (If you have such a complex structure you are more likely have several different host apps that use the module. Module maintainers are likely to have their module as a product, so they have their own goals and backlog.)

### Types of a project

There are 3 basic types your iOS project can be in according to the actual physical boundaries between your modules: monolith, modular monorepo (mono repository), and multirepo (each module in a separate repository).

![](/images-posts/2020-06-08-modularity-1-boundaries/IMG_E9635.JPG)

**Monolith**

That's a position most of the project start from. All the project's code resides in one repository without any modular separation. It's a totally valid condition when your project is relatively small and there are not so many developers working on it. Even if you have some architectural separation but without physical modules, your app is still a monolith.

**Modular monorepo**

Here your project is divided into several physical modules. All the code still resides in the same repository but there are some real borders and interfaces between the different parts. That's quite a typical setup for mega repos in tech giants like Google or Facebook.

**Multirepo**

Basically the project structure is the same as with modular monorepo: you have some host app and you have some dependencies. The only difference is that the modules have their own projects. Usually, you have to use some dependency manager to build your main project altogether. You treat your modules (which become `frameworks` or `libs`) the same way you treat 3rd party dependencies.


## False dichotomy

It's quite a common misconception that `monorepo` and  `modular project` are something completely opposite. Like in one case you have all your code in one place, no architecture, no separations of concerns - one big spaghetti bowl. In contrast, you might have all your features and core components sitting in separate repositories, perfectly encapsulated, and having no idea about each other.

Even without such extremums, this entire contraposition is still false. If it wasn't clear before this post should have evaporated this idea.

Modularity (which can be different as we described here) tells you how well different parts of your project are separated from each other. Monorepo and multirepo are the ways to store these parts, manage the changes in the codebase, and adopt these changes in the host project.

![](/images-posts/2020-06-08-modularity-1-boundaries/IMG_E9636.JPG)

## Conclusion

Boundaries are just one of the aspects of modularity. Boundaries are your safety net. The stronger the boundaries between your modules are the safer your project is in terms of keeping the separation of concerns.

&nbsp;

---
I hope you liked this piece of reading. If you have any questions, suggestions or corrections you can reach me out [on Twitter](https://twitter.com/dmtopolog)
