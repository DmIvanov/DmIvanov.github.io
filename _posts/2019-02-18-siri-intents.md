---
title: Siri Intents
date: 2019-02-18
author: topolog
layout: post
permalink: /siri-intents/
image:
  path:  images-posts/2019-02-18-siri-intents/siri-logo-1920.jpg
  thumbnail: images-posts/2019-02-18-siri-intents/siri-logo-600.jpg
tags:
  - app extension
  - iOS
  - Siri
---
Siri Intents are the cornerstone notion of all the interaction with Siri no matter if it is full-fledged SiriKit usage or Siri Shortcuts. In this post we will try to figure out what Siri Intent is, what is the difference between Domain Related Intents and Siri Shortcuts and which approach is suitable for which use cases.

I deliberately don’t post any sample code in this article, you can easily find it in various manuals out there. This post is NOT about **“how”** to do things, but about **“what”** and **“why”**.

&nbsp;

## Intents

SiriKit became open for us developers with the release of iOS 10. That let us utilise Siri in our apps.. or actually in our apps' extensions.

The core of interacting with Siri is an intent - instance of INIntent subclass. There might be some predefine intent classes (Domain Related Intents) or your custom ones (Shortcuts Intents).

Siri doesn't interact directly with your app but it does it through the app extension. An extension acts as a mediator between the user and the functionality of your app.

![](/images-posts/2019-02-18-siri-intents/user-siri-app.png)

There is host/containing app concept when working with extensions (I described it in [one of my previous posts](https://dmtopolog.com/ios-app-extensions-data-sharing/)). In case of Intents Extension Siri plays the host role. That’s the app which activates your extension and exchanges data with it. Siri sends requests/commands which it gets from the user and returns some response which your extension provides. For its requests Siri has quite small timeout: somewhat 5-10 seconds (empirically). If your extension doesn’t return a response within this time frame Siri would consider that request failed.

The app exposes its intents for the system in the \`info.plist\` file, donates them and add as suggestions to the spotlight. When the user talks to Siri she tries to figure out what app the user wants to use (if any) and which intent to create out of information the user provides. If an intent is created it's being passed to the app, and then the app is responsible for proper handling and running some tasks.

&nbsp;

## Interaction steps

Basically intent is user's **Request** to the system (or to some app). There might be different types of requests: in one case you want the system to do something for you, in the other case you want to get some information back (quite like PUT and GET http requests). Hence this requests when handled can have different types of **Response**. Some of them may just trigger some action in your code and success/failure is the only response you can provide. In the other case the essence of the request is to get some data in visual or audio form, so the response might have properties filled with some data, some spoken phrase to convey this data to the user and even custom UI to display this data. But we still have some action your code performs to get this data - let’s call it **Handling**.

If you look deeper into these three stages you will see that Request from the beginning might be quite vague, so Siri needs to clarify it with additional questions. So we also consider **Resolution** stage of the process, when Siri tries to understand what user wants. If it’s recognised that your app’s intent should be used, Siri tries to resolve all the necessary parameters of the intent.

When all the parameters resolved but before the actual handling we have an opportunity to check if we are ready for the action. We may check user’s authentication or whether or not we have an item which is about to be ordered in stock. We need to confirm that we are able to make it. We might also want some confirmation from the user when we are not sure we got everything correct (or mistakes cost a lot). So here is a **Confirmation** stage, with it’s dedicated callback to the intent handler.

So eventually we have such a chain of steps:

![](/images-posts/2019-02-18-siri-intents/steps.png)

&nbsp;

## Domain related intents

Intents are the interface which Siri uses to communicate to your app. This interface is not a free form, it's quite restricted now. Siri cannot figure out what user wants from your app if you don't give some hints in advance. So there are some predefined types of actions the user can apply to your app. These types are grouped in several categories - domains:

![](/images-posts/2019-02-18-siri-intents/SiriKit-domains.png)

For instance one of the domains has to do with making a reservation at the restaurant. In this domain you are able to check restaurant availability, book the place or check your future reservations. But I'm not sure you will be able to check which kinds of draft beer they have, are there going to be any live performance tonight or whether they allow dogs inside. Even if you want to implement this functionality in your restaurant app it’s likely to be impossible within the domain. All these domains have some predefined set of activities - concrete INIntent subclasses you can use in your app. If the activity, that you want to provide in your app, cannot be expressed within the domain or it doesn't fit any domain at all, your users are not able to use Siri for triggering it from free form conversation.

Imagine your app declares some function it supports - let's say booking a restaurant. You as a developer want to integrate this functionality with Siri so the users can use your app to make a reservation from Siri. The user needs to provide the place, the date and the amount of people for the reservation. At some point in your app you donate this intent to Siri and Siri will try to catch when the user wants to do exactly that - to book a restaurant via your app.

> Hey Siri, book me a table in Uncle Joe’s tonight


If the data from the user’s original request is incomplete (we miss the time of the reservation or the amount of people to participate) Siri will ask some clarifying questions to get this piece of information. Eventually the reservation is done and the api request is being sent to your server. All of that happened in Siri UI without launching your app.

![](/images-posts/2019-02-18-siri-intents/scheme-domain-related-intents.png)

If we try to split this use case into Resolution - Confirmation - Handling you’ll see how important is Resolution part here. Siri needs to leverage all the power of speech recognition to understand all the details of the user’s request.

In such cases like ordering something, paying for something or doing some other actions with significant consequences it’s important to make a Confirmation. For Domain Related Intents there is confirm(intent, completion) method in your designated intent handler, where Siri provides the intent it created so you can check if everything is correct and you are ready to handle this intent.

([_Here_](https://developer.apple.com/design/human-interface-guidelines/sirikit/overview/domains-and-intents/) _you can find a list of predefined intents for all the domains)_

&nbsp;

## Siri Shortcuts

These Domain Related Intents are really powerful, but you can utilise this power only if the functionality you provide belongs to one of the predefined domains. I can imagine which technical restrictions caused that but the fact was that lots of applications couldn’t benefit from SiriKit before the release of iOS 12.

ffThese Domain Related Intents are really powerful, but you can utilise this power only if the functionality you provide belongs to one of the predefined domains. I can imagine which technical restrictions caused that but the fact was that lots of applications couldn’t benefit from SiriKit before the release of iOS 12.

For instance, we maintain the app which is a social network for stamp collectors. You as a user have your own profile inside the app where you keep track of all the stamps you collected. You also have some friends in the app who also collect stamps, and you can open their profiles and take a look at stamps they've added.

Let's say we want to let the user an ability to interact with Siri to check what stamps their friends have recently added to their collection. The user just asks:

> Hey Siri, what is the last stamp Mark added?


Saying that you expect Siri to tell you the name and the year of the stamp. You might also want Siri to show you the actual stamp picture.

What would you do?

First of all you'll check all the existing domains trying to adjust your functionality for one of them. If there are built-in intents which can be used you should pick them. That gives you fully fledged Siri experience. For our stamp app we cannot use any of the domains so Siri Shortcuts is our way to go.

![](/images-posts/2019-02-18-siri-intents/scheme-siri-shortcuts.png)

On one hand with the shortcuts you as a developer can implement whatever functionality you want without any restriction. On another hand your users are more restricted in the way they can reach this functionality. With the Domain Related Intents the user doesn’t predefine anything - all work is done by Siri and your code. The user just talks to Siri in a free form naturally (like to a friend) explaining what they want. In case of the shortcut you have an intent - basically the same subclass of INIntent - but the parameters of this intent should be prefilled, they cannot be figured out by Siri in realtime. In both cases Siri interacts with the user and pass an intent object to your app for handling. But with Domains you donate an _intent class_ and Siri creates an instance of this class with all the properties. With the shortcuts you originally donate _a set of instances_ of your intent class to the system and Siri just does the job to pick the proper one.

Back to our stamp social network. The developer precisely defines a separate intent for each friend the user has in our app. If the user has 10 friends, 10 intent object will be donated to Siri. We just need the user to mention the exact friend so Siri can pick up and pass this friend’s intent to our code and then we can provide requested stamp info.

In case of shortcuts Siri doesn't do any AI work trying to get an intent from the user, so there is no Resolution step in the process. The intent is already donated by the app with all the parameter's filled in and connected to some specific phrase (or set of phrases). So Siri's job is just speech recognition to get the phrase. Phrase is linked to the intent which is the actual request in this case, so it's being passed to the app extension. The extension fetches the data - does the action - and provides it back to the user as a response.

We create \`.intentdefinition\`-file to specify our custom intent classes which we use for shortcuts. When you create a custom intent an accompanying response class is being created automatically for you. In the definition file you specify the type of your intent, the type of the response, what types of result you may return and how Siri should react on these results. You can even specify different types of errors which may occur in the process of interaction with your intention.

On the request tab of the intent definition we have a special checkbox which is responsible for whether or not the user needs to confirm the action which supposed to be provided by your code. That’s what we have for Confirmation step for this type of intents. If you pick Order, Book or Buy type of the intent the confirmation is obligatory for the user.

It’s not necessary to use Intent Extension to utilise Siri Shortcuts. You can do it easier via NSUserActivity. But such shortcuts are only able to open your app and pass the NSUserActivity-object there. No custom responses, no custom UI.

&nbsp;

## Donation

When we talk about Shortcuts we have an additional element in chain of steps (Request - Resolution - Confirmation - Handling - Response) and it’s **Donation**.

When we use SiriKit for Domain Related Intents we just add out supported intents into IntentsSupported array of `info.plist` file. As all the intent subclasses are default in this case, just having a name of a concrete class is enough for the system to figure out the intent structure with all the properties.

As I mentioned earlier, with the Shortcuts we pass instances but not classes to iOS. Hence we have to not only provide all the intent structure, but an entire instantiated object. So we donate an intent only when we have all the data to fill its properties. This passing an object to Siri is called [“donation”](https://developer.apple.com/documentation/sirikit/donating_shortcuts).

But there is another side of the process which makes the actual moment of donation much more important - that’s [Siri Suggestions.](https://developer.apple.com/documentation/sirikit/shortcut_management/suggesting_shortcuts_to_users)

Siri analyses when the shortcut is being donated to make an assumptions when to suggest this shortcut to the user in future. You have to donate a shortcut exactly at the time when user uses the functionality which the shortcut is related to (or at least opens a corresponding screen). So if the shortcut lets the user to order soup you have to donate it every time the user does this action in your app. Let’s say it’s being done every day about 1pm, so eventually Siri will figure out that that time is the right moment to suggest an appropriate shortcut to the user. It can do it in places such as Spotlight search, Lock Screen, and the Siri watch face.

That’s why it’s extremely important to pick some key functionality of the app for the shortcut. If the user needs this feature only once in a while, Siri will know it from the frequency of the donations and hardly ever will suggest it. But if using properly it can enhance the entire user experience in using your app.

I mentioned for Domain Related Intents you don’t need the donation. But you can still use it (together with adding an intent class into your intent definition file) to make it available for Siri to predict and suggest it.

&nbsp;

## Use cases

So let’s summarise all that was said and try to match specific Siri related technology with what we (as developers) want to implement.

In the easiest case we might want the user to be able to just open the app and route to some particular screen. This scenario is similar to using a deep link. Usually the link is triggered via push notification or opening some URL with our app as a target, but here we use some custom phrase (chosen by the user) and then parse it accordingly. In the app we don’t need to add Intents extension for that. We just define an intent, create it and donate to the system using NSUserActivity. Then we just catch when the app is being launched by this user activity and properly process it - routing to some specific place, showing some particular data. This way we provide a primitive shortcut for the user.

If we want more complex integration we check what can Domain Related Intents offer to us. If our app and the functionality we want to implement fit some of the domains we choose this path. Here we get free form communication between the user and Siri where Siri tries to clarify for us what the user wants. Eventually we should handle an intention Siri created and provide some response. If we need some custom UI we are able to create it for the response model and pass it to Siri. For implementing that we need to pick some predefined intent, add an Intents Extension (and optionally an Intents UI Extension), and implement some handlers there. If we provide some custom UI to Siri the user may tap on it so we might want to implement some deep routing here as well (as we did in the previous paragraph).

In case when we need an extensive integration with Siri but we cannot find any predefined intent for our functionality, we go for Siri Shortcuts. In this scenario we loos the ability for the user to have a free chat with Siri. Instead of that we ask them to created a specific phrase which we then bind to the intent. When intent is handled we have to process it and provide some response (of a structure we defined ourselves). Custom UI is also an option we can use. Of course we need to adopt Intent Extensions for that.

If we want Siri to suggest our functionality to the user (regardless the approach we’ve chosen) we add our intent to the intent definition file and donate an instance every time the user does the action in the app. Eventually Siri will figure out when is the best time and place to propose this function to the user.

&nbsp;

---
I hope you enjoyed this piece of reading. If you have any questions, suggestions or corrections you can reach me out [on Twitter](https://twitter.com/dmtopolog)
