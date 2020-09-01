---
title: Thoughts on rewriting the Messenger App by Facebook.
date: 2020-03-04
author: topolog
layout: post
permalink: /fb-rewriting-messenger/
image:
  path: images-posts/2020-03-04-fb-rewriting-messenger/twits-2800.png
  thumbnail: images-posts/2020-03-04-fb-rewriting-messenger/twits-600.png
tags:
  - iOS
  - architecture
  - thoughts
  - no code
share: true
---

In my small iOS information bubble [this post from FB-engineering](https://engineering.fb.com/data-infrastructure/messenger/) exploded so nobody could ignore it.

The funny thing is that people look to this article like into a mirror and everybody sees what they want to see. Some people blame FB for a waste of resources, the others praise them for being bold, switching to a new architecture and putting performance first. Some folks start the funeral for ReactNative the others find it as the prove that cross-platform (Flutter or even Kotlin Multiplatform) is our nearest future. That's really awesome!

I also wanted to tweet something while I was reading it but eventually I had several thoughts (and interpretations) which I wanted to write down.

First of all it's a very compelling reading because indeed there are not so many stories about such big projects. I personally cannot remember one in our iOS world since Airbnb and [their experience of taking up and throwing away React Native.](https://medium.com/airbnb-engineering/react-native-at-airbnb-f95aa460be1c).

I used to work quite extensively with the real-time messaging app (web-sockets, high load, async UI) and the one which heavily relied on DB-to-backend data syncing (fancy CoreData context models, dozens of tables, millions of data entries, several simultaneous data streams). It was two different projects and both of them was full of technical challenges. FB has both in one app, which they managed to redo from scratch. That's something to appreciate for sure. If you want to say "it's only a matter of resources", believe me it's not.

The post is definitely written in collaboration between the engineers and the marketing people. The language is quite generic and there are lots of vague constructions. But at the same time it's full of technical details, chosen approaches and the attitude to some FB-maintained technologies. They managed to say a lot without saying it explicitly. So people will keep on interpreting it in all the possible ways.

I'm really surprised how many people treated it as the beginning of the end for RN. [This small thread](https://twitter.com/dan_abramov/status/1234801507805138945) from [Dan Abramov](https://twitter.com/dan_abramov) - one of the leading guys in RN-team - nails it I think.

Nobody in FB ever said that RN is a universal tool for all the problems. It would be weird to consider that FB plans to move everything to RN and stop doing native development (especially if you know how fast their native teams grow and how much they invest into compilers and other low level native tools).

People focus on React native and nobody seems to remember ComponentKit, async UI and the declarative paradigm? Is it something which made sense 5-10 years ago (and were likely to be used in an old Messenger app) but looks more as a burden now.

> While UI frameworks can be powerful and increase developer productivity, they require constant upkeep and maintenance to keep up with the ever-changing mobile OS landscape. Rather than reinventing the wheel, we used the UI framework available on the device’s native OS to support a wider variety of application feature needs.

That pretty much explains everything. Nowadays in many cases the benefits some custom fine-tuned solution give you are smaller than the headache you have maintaining them. It's always a trade-off: whether to pick an existing general solution or to reinvent a wheel which would perfectly fit your needs. No development and maintenance overhead against your custom use cases and specific requirements. When it comes to custom alternatives for system frameworks, there are some additional points to consider. System frameworks grow bigger covering more and more edge cases and encapsulating more and more functionality. Moreover in case of iOS when you use some system components you are relatively protected from the future changes and API updates. So the choice is getting more obvious.

BUT what works for some cases doesn't work for the others. And you still might need to write some C components replacing the native libs for performance or create your own layer atop of SQLite.

About that new SQLite wrapper.. I heard it's quite a revolutionary engine (would be interesting to see more details about it). There is an idea that this MSYS is a future core components for all the major FB products (main app, Instagram, WhatsApp). It makes sense considering how much effort did they invest into it. I don't believe such an effort (several years and 100 developers) can take place only because the old app was such a crap or because they need to occupy the engineers with some interesting big task. Seems like Messenger is a pilot (used by millions of people.. but still not the main product) for the company to test some new technologies and approaches, which will shape and improve the future of the entire company? How else you can sell such a project to your managers? ;-)

This story also made me think about the trade-offs of big companies between huge resources and lack of flexibility. What was the state of the art 6-8 years ago is dusty legacy now, but you cannot easily get rid of it or migrate to a new technology. At that times there were no architecture in iOS except MVC, not so many people actively used patterns, DRY, KISS and all the other basic principles, "reactive" and "declarative" were not parts of a developer's vocabulary. I can imagine some leftover from that wild-west times. You can still find tons of code to satisfy some requirements which make no sense now, code you cannot easily refactor or remove. Building from scratch might be a good idea in this case (which is not for majority of the other (normal) cases)

The last thing. While reading I was eager to find some mentioning of Swift. But... custom C components... SQLite wrapper... 'categories' of basic views... So the same C-based stack as before for the new app in 2020. I know FB has reasons and I do understand most of them, I really like ObjC for its freedom and runtime capabilities... I think I'm just awaiting some new era which starts when FB deploys the first swift code into their main app ;-)
