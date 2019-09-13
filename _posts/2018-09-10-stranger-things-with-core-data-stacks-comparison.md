---
id: 36
title: Stranger things with Core Data Stacks comparison
date: 2018-09-10T17:29:49-02:00
author: topolog
layout: post
guid: https://dmtopolog.com/?p=36
permalink: /stranger-things-with-core-data-stacks-comparison/
image:
  path: images-posts/2018-09-10-stranger-things-with-core-data-stacks-comparison/hero-2000.jpg
  thumbnail: images-posts/2018-09-10-stranger-things-with-core-data-stacks-comparison/hero-600.jpg
  caption: Photo by Puneeth Shetty on Unsplash
categories:
  - Tech Blog
tags:
  - CoreData
  - iOS

---

Core Data is one of the most arguable frameworks for iOS/macOS development and Core Data Stack is indeed one of the most arguable questions inside Core Data topic. Which stack to choose? Which one is more efficient? Discussions about all that have been going on in the community for years.

Here are the most common stacks:

![](/images-posts/2018-09-10-stranger-things-with-core-data-stacks-comparison/Stacks-all-together.png)

There were several different experiments and several points of view about which one is better. Some people including Marcus Zarra promoted Nested Contexts as the best option for all cases. Florian Kugler from ObjC.io in his famous post “Concurrent core data stacks — performance shootout” back to 2013 measured performance and compared Nested Contexts with classical Shared Coordinator stack (his entire blog is no longer exists at <http://floriankugler.com/blog> but you can still find that article in some internet cache). He developed that investigation and his Core Data book stated again that Shared Coordinator is the most suitable approach, because it doesn’t affect the main thread as much as Nested Contexts model does.

And it looked like with the release of NSPersistentContainer in 2016 all the arguments should stop. Apple presented their vision on the matter (which was aligned with what they were talking before on Core Data sessions at WWDC). And indeed it’s quite a convenient way to go which doesn’t require all that boilerplate code and answers all the stack-related questions instead of you.

NSPersistentContainer is a variant of the good old Shared Coordinator model: there are one Coordinator, one static main context which is created together with the container and multiple worker background contexts which are created on demand. All the contexts have the coordinator as their parent (no nesting contexts) and can subscribe to each other’s changes if you need a synchronisation.

I decided to check out how good is NSPersistentContainer comparing to all the other options. But when I looked for any investigations about that (common… 2 years have passed!), any performance comparison I couldn’t find anything. So I had nothing to do but to spend some time myself.

So I constructed all the basic stack models, ran some fundamental operations against them and measured the performance with Profiler. I did:
  - Background insertion. I inserted 10k objects into an empty data base (of course saving the context after each one.. of course, that’s not how you supposed to do in real project)
  - Background insertion together with simultaneous fetch from the main context (inserting 10k objects, fetching 10k other objects)
  - Fetch from the main thread just to see how much pure time it takes.

The sequence of the actions was:
  1. Fresh installation
  2. DB population (objects with numbers 0…9999)
  3. Fetching (objects with numbers 0…9999)
  4. DB population (objects with numbers 10000…19999) and fetching (objects with numbers 0…9999)

Here is my result:

![](/images-posts/2018-09-10-stranger-things-with-core-data-stacks-comparison/Stacks-Performance-Table-2.png)

Basically everything is clear:
  - Simple Stack sucks!
  - Nested Contexts loads the main thread more that Shared Store or Shared Coordinator
  - NSPersistentContainer has the same performance as Shared Coordinator (a bit better when merging changes between context, a bit worse without merging)
  - Nested Contexts is great if you need to insert objects, but not necessarily need to propagate the changes down to disc.

BUT… I think you’ve already noticed an elephant in the room..

What’s going on with Nested Contexts without changes propagation in case of fetching data?

In theory it should be fast if we have all the objects in Main MOC and just retrieve them from it without getting to the storage. But in my experiments it took ages. I tried to figure out what’s wrong with my setup, I changed the way I did the query (removed the sorting, made it completely synchronous, removed recursion,…) nothing changed the result much. From what I understand in this case (fetching data which is already in the context) there should be no difference if the data is stored on disc or not. But the Nested Contexts stack with changes propagation behaves without any surprises.

The second question is why NSPersistentContainer (and his older brother Shared Coordinator) are slower than Nested Contexts in simultaneous fetching and storing the data. In my case data insertion for Nested Contexts takes about 5 extra seconds of main thread work. But in combination with fetching Nested Contexts performs much better and it takes almost the came amount of time, when for NSPersistentContainer it’s about the sum of fetching time and insertion time all together. How come? The mystery or some flaws in my experimental setup?

After couple of days full of thoughts and experiments a gave up, and decided to write this post asking for help. So if anyone could enlighten me I would be more than appreciate it.

([Here is the setup I worked with](https://github.com/DmIvanov/CoreDataStacks))

You can get some more insights into Core Data stacks, their differences and some experience migrating from one stack to another from [my presentation at CocoaHeadsNL](https://youtu.be/BF9DD8XXxaI)
