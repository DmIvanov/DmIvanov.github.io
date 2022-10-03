---
title: Modularity. What problem does it solve?
date: 2022-10-03
author: topolog
layout: post
permalink: /modularity-3-problems
image:
  path: images-posts/2022-10-03-modularity-3-problems/nails-1920x620.jpg
  thumbnail: images-posts/2022-10-03-modularity-3-problems/nails-600.jpg
  caption: Photo by topolog
tags:
  - modules
  - iOS
  - architecture
  - no code
categories:
  - Tech Blog
share: true
---


In this article we will discuss the problem that can be solved by modularity and how exactly modularity can make you project much better thing to deal with.

This post is part of the series on modularity:
- [Modularity. Boundaries](/modularity-1-boundaries)
- [Modularity. Encapsulation](/modularity-2-encapsulation)
- [**Modularity. What problem does it solve?**](/modularity-3-problems)

&nbsp;

# When does the time come?

When is it right time to think about modularization in your project? 

It doesn't look like you should divide your project into the modules from the very creation. You definitely need to follow all the major architectural principles (separation of responsibilities, clear dependencies, architectural layers, DRY, KISS etc.) when building your logic, but you are likely not able to foresee how the project will look like in a year or even several months. Your business ideas, direction of development, target audience, your main features... all may change. At the beginning of its existence, your app can turn into something completely different within several months. So, it doesn't make sense to start dividing and incapsulating high-level parts of business logic at this point. If you start to design your modules and interactions between them too early, you will be constantly wasting time on restructuring and changing.

The other extremum is when the situation has already gone so far that you need to stop (or at least significantly slow down) development of your features to modularize the project. At this point you might already have more than a dozen of developers, serious commitment to deliver features, ambitious goals... but the project is a mess, technical debt is enormous, bugs are all around and releasing new features gets harder and harder. If you found yourself here, then you should have already modularized your project some time ago. I'm not saying that now it's already "too late", no, but it costs you much more troubles to do it now than before.

As always, the truth is somewhere in between, and of course it's not the same for different teams and different projects.

### Making the terms clear

First let's clarify what we mean when you say that the project is **a monolith**.

It might have different dependencies, some of them might be your own modules or subprojects. You might already have some separated modules for common data types, or design components, or network layer. You might have several different targets for extensions (watch, Siri, widgets, etc.). BUT more than 50% of your logic belongs to one "main" target (you can roughly count the number of lines of code together with the number of classes/functions). So, you have a monolith.

When we say that the app/project is **modular** we mean that it consists of a number of modules (separate targets inside one project or one main project and several dependent ones). All the code is spread across those separate logical parts and there are strict physical boundaries between them (read more about the boundaries and different types of projects in [our previous article](/modularity-1-boundaries)).


# Reasons to think about modularization.

Let's try to figure out some concrete signs of the project that requires modularization. If your project has some of them it's likely the time to modularize it.

The most popular reason for moving some code to a separate modules is situation when you need to share some logic between several apps. It might be network layer, common data types, functions or UI elements. We omit it here as it's quite intuitive and doesn't require much explanation. But there are more than that.

### Poor logical separation

At some point your project becomes so big and the flows are so complex that there is not a single developer who understands how all the features work. Plenty of features are not a compilation per se, but quite often those features are poorly separated architecturally. It's really hard to measure such "architectural health" but if you start asking yourself the following questions you are likely to understand how good/bad the situation is.
- Do you have clear boundaries between the screens, flows and features?
- Do you clearly understand the dependencies of the features?
- How easy it is to use your features/flows in a different context (in a different part of the app)?
- How do you handle shared functions between the features? 
- Do you have any incapsulated services for networking, persistence, analytics, logging, error handling... or maybe you put all of these to the extensions (not the best approach)? 
- Do you have a lot of data types that don't really belong to anything? 
- Do you use Dependency Injection? 
- How many singletons do you have? 
- How often do you prefer to use a singleton instead of passing a dependency explicitly?
etc.

One of the bad indicators is situations when you try to grasp the context of an unfamiliar feature/service... but you just can't. It's not properly separated from the others; you cannot easily figure out all of the dependency. When tracking down the behavior in the debugger you keep on jumping from one place to another between separate classes and even parts of the app. 

The more features you have, the more obvious your architectural imperfections are.

### A lot of conflicts

Not only conflicts in pull requests matter, but generally how often one developer blocks another in their daily work. Starting from a team of 2 developers you need to somehow synchronize the work and coordinate the effort. With a badly architectured app and some decent number of features it can be painful for just 2 devs to work together. If the health of the code base is better, then even 3-5 developers can work on one monolithic project with a decent (though not the best) efficiency. But the more the team grows the more time on coordination you need to spend. Deliveries never grow proportionally to the size of team (twice more developers are never able to deliver twice more features), but if the amount of deliveries hardly increases when scaling the team that's a bad sign. 

The chances are that you do have some separation of logic and areas of responsibility for developers. Maybe you even have some feature teams dedicated to some specific app parts. But ask yourself how clear the boundaries between them are. How much code reside in the "gray areas" which don't belong to anything? How often it's not clear who supposed to work on this or that piece of logic? Or the other way around: there are some parts of the app that are constantly being changed by different developers (because many features/services depend on it).

At some point (sooner than later) constant conflicts and lack of clarity may cause serious frustration in the team (which, as we know, decreases satisfaction level and productivity and can even cost you some team members)

### Compilation speed is unbearable

If you have a big monolithic target you need to recompile it entirely after every small change. As far as I understand the compiler still tries to make some smart chooses to decrease the compilation time (more with ObjC code then with Swift), but it doesn't significantly change the situation. Hence when debugging the code, changing and rebuilding it all the time takes an eternity (can easily reach an hour or two cumulatively during the day). You frequently distract yourself, read articles, browse social media or do something else, so the efficiency drops even lower.

### Re-running all the tests

Quite likely if your project is not modularized your test coverage is not so high. Testability and modularization are not strictly connected, but there is a correlation. But let's imagine that the architectural health is decent in your project, and even if it's a monolith the code is testable and different parts are logically separated from each other. So, it is possible to have a good test coverage. Let's keep on dreaming and consider you have it. In this case you need to run all the tests (locally or on CI) for every little change. And as the codebase (and test coverage) get bigger it becomes more and more frustrating (hundreds of UI tests take tens of minutes).

I heard about some smart solutions when people create some functionality clusters and per each change define what part of the tests should be rerun. But in a monolith, it's just an overkill and waste of resources.

---

The points above are quite subtle, there are no clear metrics or specific app qualities that you can monitor. It also usually hard to sell it to business to get time for the necessary refactoring. But if you proceed thinking further you can easily see how they affect the main properties of your product.

### Decreasing project reliability

It's a combination of bad architectural separation and code base growth (not talking about general code quality). When you have "spaghetti code" where everything depends on everything it's hard to change something without breaking something else. You might notice that the number of bugs, crashes and other side effects increases. That quickly influences your users, their satisfaction and opinion about your app (the rating, in-apps or sales inside the app logically go down).

### Decreasing efficiency

You spend more time on the maintenance and fixes, more time to synchronize work of different developers, more time on compiling, debugging and testing the code, slow CI pipelines together with high probability of merge conflicts make every merge long and painful, it takes hours and even days to figure out how things work inside an unfamiliar piece of logic... Eventually the development of the new features stalls. If you delivered several features per week/month/sprint before, now this amount noticeably decreased. It's hard to stick to the planning and meet the deadlines because issues pop here and there all the time.

Maybe you already hear such complaints from the business.

# How does modularization help you?

I'm not saying that all the issues will be magically fixed if you modularize your project. Each of the issues above normally has several reasons and all cases are different. You definitely need to also think about using advanced architectural patterns and best practices, apply some code guidelines and QA-tools, check your work processes and the informational channels inside the team. But I'm sure that modularization will help you to improve the situation in all those aspects.

On a high-level, modular project is more structured than the monolith. Module is a separate unit, an extra layer of abstraction. When working on the host app (outside the module) we deal with modules instead of vague "parts" or "layers" that didn't have clear responsibilities and boundaries. Specific data types on this level become just implementational details, so we can cut these details off. When working inside the module we don't need to think about the context of the entire app, it doesn't matter anymore while the module keeps its interface stable. So regardless of your context you decrease the cognitive load by reducing the context.

Modules are independent projects (even when having dependencies from each other) so the scope for the work processes also decreases. We are talking about several small project instead of one big monolith, hence we have independent, much faster development loops (develop, test, release). We have smaller number of features and much less conflicts: now instead of 10 devs working on the same monolith we have 5 teams of 2 devs working each on independent module.

_(More details about context separation, and modules as independent projects you can read [here](/modularity-2-encapsulation))_

We don't have problems with long compilation and lengthy test runs anymore, as all of these is specific to the module. (You still need to debug and test the integration code, but the amount of work decreases several fold).

In big teams (when more than 4-6 devs work on one app) organizational structure normally changes together with the project. Modules work better when they have clear maintainers. So, the next logical step might be to establish some feature teams with clear areas of responsibility.

No-brainer that because all of this the overall productivity and satisfaction increases because generally it just gets easier to work. The prerequisite-consequence connection here is quite straightforward.

Regarding the reliability, reducing both the context and the connections between the parts of the app (and side effects as a result) changes the situation dramatically (in a good way of course =)). Code changes become easier to make and test, issues become easier to localize and debug.

_(It was a high level overview of the benefits. In future posts I plan to talk more in details about each one of them: scalability, compilation, work processes etc. mentioning also the challenges that modularisation brings)_

## Bonus improvements

I already mentioned that there are a number of things that influence the maintainability and reliability of your code that are not directly related to modularization. Some of them are also got improved on the way together with modularization. Some changes are inevitable side effect of the refactoring needed for modularization. Some other improvements just look more logical in a new modular context. The others were just waiting in the backlog for ages and transition to the modular architecture is a good occasion to fix it on the way.

Some examples:

Usage of **dependency injection** and less **singletons** will come naturally together with modularization. When you don't have implicit shared state between the modules you are forced to pass the dependencies explicitly. Inside the modules you can still avoid it and use the dark side of the force (singletons and internal instantiation of dependencies), but you are likely to start questioning this approach as it is not a default approach. If you know all about DI and singletons and just were waiting for the occasion to refactor some legacy shared state (which usually requires quite some changes all around the app), migration to the modules is the time to do it.

You might also rethink some responsibilities in your code (as now you need to draw bold lines between them), on the way you might refactor a couple of **god classes** the ones that have couple of thousands of lines of code and a bunch of different responsibilities.

Modularity itself and advanced architectural techniques will increase **testability** of your code.

When designing your modules, you normally would discuss inside the team some general architectural patterns and approaches that your future modules supposed to share. If you go further, you may even write down some guidelines on how the future modules should be created, structured and maintained. That will bring more **architectural and/or code style consistency** to your code base that increases the productivity even more in future.

You can refactor some small pieces of the **legacy ObjC code** that just don't fit your new modular frame of the project.

Of course, you should understand that making several migrations on the same time is not a right thing to do. As well as fixing too much of technical debt simultaneously. So, if you have some small things here and there, go ahead and fix them on the way along with modularization. But if the side fixing of something requires too much additional effort, please put it aside and pick it up later when modularization is over. I know how difficult it can be and how big is the temptation to cure all the diseases at once. I'm sure you will have enough work to do with modularization only, so don't unnecessary increase the scope even more. All your dirty legacy will be cut into pieces and nicely split into modules together with the rest of your code, so it will be anyway easier to handle it after the modularization.

&nbsp;

---
I hope you liked this piece of reading. If you have any questions, suggestions or corrections you can reach me out [on Twitter](https://twitter.com/dmtopolog)

