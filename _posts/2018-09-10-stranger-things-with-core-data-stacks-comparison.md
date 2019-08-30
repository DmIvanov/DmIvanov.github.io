---
id: 36
title: Stranger things with Core Data Stacks comparison
date: 2018-09-10T17:29:49-02:00
author: topolog
layout: post
guid: https://dmtopolog.com/?p=36
permalink: /stranger-things-with-core-data-stacks-comparison/
categories:
  - Tech Blog
tags:
  - CoreData
  - iOS
---
<p id="424f" class="graf graf--p graf-after--figure">
  Core Data is one of the most arguable frameworks for iOS/macOS development and Core Data Stack is indeed one of the most arguable questions inside Core Data topic. Which stack to choose? Which one is more efficient? Discussions about all that have been going on in the community for years.
</p>

<p id="143c" class="graf graf--p graf-after--p">
  Here are the most common stacks:
</p>

<img class="alignnone wp-image-37" src="https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/10/Stacks-all-together.png?resize=688%2C259&#038;ssl=1" alt="" width="688" height="259" srcset="https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/10/Stacks-all-together.png?resize=300%2C113&ssl=1 300w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/10/Stacks-all-together.png?resize=768%2C288&ssl=1 768w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/10/Stacks-all-together.png?resize=1024%2C384&ssl=1 1024w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/10/Stacks-all-together.png?w=1600&ssl=1 1600w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/10/Stacks-all-together.png?w=1376&ssl=1 1376w" sizes="(max-width: 688px) 100vw, 688px" data-recalc-dims="1" /> 

<p id="bc85" class="graf graf--p graf-after--figure">
  There were several different experiments and several points of view about which one is better. Some people including Marcus Zarra promoted Nested Contexts as the best option for all cases. Florian Kugler from ObjC.io in his famous post “Concurrent core data stacks — performance shootout” back to 2013 measured performance and compared Nested Contexts with classical Shared Coordinator stack (his entire blog is no longer exists at <a class="markup--anchor markup--p-anchor" href="http://floriankugler.com/blog," target="_blank" rel="noopener nofollow" data-href="http://floriankugler.com/blog,">http://floriankugler.com/blog,</a> but you can still find that article in some internet cache). He developed that investigation and his Core Data book stated again that Shared Coordinator is the most suitable approach, because it doesn’t affect the main thread as much as Nested Contexts model does.
</p>

<p id="57bc" class="graf graf--p graf-after--p">
  And it looked like with the release of NSPersistentContainer in 2016 all the arguments should stop. Apple presented their vision on the matter (which was aligned with what they were talking before on Core Data sessions at WWDC). And indeed it’s quite a convenient way to go which doesn’t require all that boilerplate code and answers all the stack-related questions instead of you.
</p>

<p id="b720" class="graf graf--p graf-after--p">
  NSPersistentContainer is a variant of the good old Shared Coordinator model: there are one Coordinator, one static main context which is created together with the container and multiple worker background contexts which are created on demand. All the contexts have the coordinator as their parent (no nesting contexts) and can subscribe to each other’s changes if you need a synchronisation.
</p>

<p id="b61d" class="graf graf--p graf-after--p">
  I decided to check out how good is NSPersistentContainer comparing to all the other options. But when I looked for any investigations about that (common… 2 years have passed!), any performance comparison I couldn’t find anything. So I had nothing to do but to spend some time myself.
</p>

<p id="a328" class="graf graf--p graf-after--p">
  So I constructed all the basic stack models, ran some fundamental operations against them and measured the performance with Profiler. I did:
</p>

<p id="e93c" class="graf graf--p graf-after--p">
  &#8211; Background insertion. I inserted 10k objects into an empty data base (of course saving the context after each one.. of course, that’s not how you supposed to do in real project)
</p>

<p id="0db9" class="graf graf--p graf-after--p">
  &#8211; Background insertion together with simultaneous fetch from the main context (inserting 10k objects, fetching 10k other objects)
</p>

<p id="a71f" class="graf graf--p graf-after--p">
  &#8211; Fetch from the main thread just to see how much pure time it takes.
</p>

<p id="2ada" class="graf graf--p graf-after--p">
  The sequence of the actions was:
</p>

<p id="9fe5" class="graf graf--p graf-after--p">
  1. Fresh installation
</p>

<p id="6efa" class="graf graf--p graf-after--p">
  2. DB population (objects with numbers 0…9999)
</p>

<p id="a070" class="graf graf--p graf-after--p">
  3. Fetching (objects with numbers 0…9999)
</p>

<p id="4230" class="graf graf--p graf-after--p">
  4. DB population (objects with numbers 10000…19999) and fetching (objects with numbers 0…9999)
</p>

<p id="75f1" class="graf graf--p graf-after--p">
  Here is my result:
</p>

<img class="alignnone wp-image-39" src="https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/10/Stacks-Performance-Table-2.png?resize=688%2C472&#038;ssl=1" alt="" width="688" height="472" srcset="https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/10/Stacks-Performance-Table-2.png?resize=300%2C206&ssl=1 300w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/10/Stacks-Performance-Table-2.png?resize=768%2C528&ssl=1 768w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/10/Stacks-Performance-Table-2.png?resize=1024%2C703&ssl=1 1024w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/10/Stacks-Performance-Table-2.png?w=1450&ssl=1 1450w, https://i0.wp.com/dmtopolog.com/wp-content/uploads/2018/10/Stacks-Performance-Table-2.png?w=1376&ssl=1 1376w" sizes="(max-width: 688px) 100vw, 688px" data-recalc-dims="1" /> 

<p id="b17a" class="graf graf--p graf-after--figure">
  Basically everything is clear:
</p>

<p id="e039" class="graf graf--p graf-after--p">
  &#8211; Simple Stack sucks!
</p>

<p id="e997" class="graf graf--p graf-after--p">
  &#8211; Nested Contexts loads the main thread more that Shared Store or Shared Coordinator
</p>

<p id="20c7" class="graf graf--p graf-after--p">
  &#8211; NSPersistentContainer has the same performance as Shared Coordinator (a bit better when merging changes between context, a bit worse without merging)
</p>

<p id="7534" class="graf graf--p graf-after--p">
  &#8211; Nested Contexts is great if you need to insert objects, but not necessarily need to propagate the changes down to disc.
</p>

<p id="da76" class="graf graf--p graf-after--p">
  BUT… I think you’ve already noticed an elephant in the room..
</p>

<p id="7fa2" class="graf graf--p graf-after--p">
  What’s going on with Nested Contexts without changes propagation in case of fetching data?
</p>

<p id="c6c9" class="graf graf--p graf-after--p">
  In theory it should be fast if we have all the objects in Main MOC and just retrieve them from it without getting to the storage. But in my experiments it took ages. I tried to figure out what’s wrong with my setup, I changed the way I did the query (removed the sorting, made it completely synchronous, removed recursion,…) nothing changed the result much. From what I understand in this case (fetching data which is already in the context) there should be no difference if the data is stored on disc or not. But the Nested Contexts stack with changes propagation behaves without any surprises.
</p>

<p id="bcc7" class="graf graf--p graf-after--p">
  The second question is why NSPersistentContainer (and his older brother Shared Coordinator) are slower than Nested Contexts in simultaneous fetching and storing the data. In my case data insertion for Nested Contexts takes about 5 extra seconds of main thread work. But in combination with fetching Nested Contexts performs much better and it takes almost the came amount of time, when for NSPersistentContainer it’s about the sum of fetching time and insertion time all together. How come? The mystery or some flaws in my experimental setup?
</p>

<p id="7ac7" class="graf graf--p graf-after--p">
  After couple of days full of thoughts and experiments a gave up, and decided to write this post asking for help. So if anyone could enlighten me I would be more than appreciate it.
</p>

<p id="60c2" class="graf graf--p graf-after--p">
  (Here is the setup I worked with: <a class="markup--anchor markup--p-anchor" href="https://github.com/DmIvanov/CoreDataStacks" target="_blank" rel="nofollow noopener" data-href="https://github.com/DmIvanov/CoreDataStacks">https://github.com/DmIvanov/CoreDataStacks</a>)
</p>

<p id="9d41" class="graf graf--p graf-after--p graf--trailing">
  You can get some more insights into Core Data stacks, their differences and some experience migrating from one stack to another from <a href="https://youtu.be/BF9DD8XXxaI">my presentation at CocoaHeadsNL</a>
</p>