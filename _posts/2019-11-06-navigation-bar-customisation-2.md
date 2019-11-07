---
title: Navigation Bar Customisation. Part 2 - UINavigationBarAppearance and proper view/model separation.
date: 2019-11-06
author: topolog
layout: post
permalink: /navigation-bar-customisation-2/
image:
  path: images-posts/2019-10-26-navigation-bar-customization/navibar-ui-inspector-1660.png
  thumbnail: images-posts/2019-10-26-navigation-bar-customization/navibar-ui-inspector-600.png
tags:
  - UI/UX
  - UINavigationController
  - iOS13
---

Navigation bar customisation is quite a trivial task, but even before iOS 13 you could approach it in couple of different ways. This year another API has appeared. The new way suppose to replace the old ones and solve the issues which were not addressed by Apple so far. Let's see how it fits into the toolset that is in our disposal.

In [a previous post of this series](http://www.dmtopolog.com/navigation-bar-customization/) we mentioned the basics of navigation bar customisation. But just you implementing this code is just a half of a deal. Another thing is how to properly structure it. How to encapsulate the code so all the customisation happens in one place, it's easy to reuse it and it's flexible in case of changes and restyling.

Let's see what are the different approaches here.

### Incapsulating the customisation

For the beginning most of the customisation code can be put in one place where you create the navigation controller:

```swift
let navigationController = UINavigationController(rootViewController: yourRootViewController)

navigationController.navigationBar.isTranslucent = false
navigationController.navigationBar.barTintColor = UIColor.white
navigationController.navigationBar.tintColor = UIColor.warm
navigationController.navigationBar.titleTextAttributes = ViewController.titleTextAttributes
navigationController.navigationBar.shadowImage = UIImage()

window?.rootViewController = navigationController
```
This approach is not scalable at all which gets clear after the second instance of UINavigationController. If you want to reuse the same code you should put it to a convenient common place.

One option is to create some factory/builder to apply a specific style to an instance of the class:
```swift
func createDarkNavigationController(rootViewController: UIViewController) -> UINavigationController {
  let navigationController = UINavigationController(rootViewController: rootViewController)
  // apply all the specific customisation
  return navigationController
}
```
Every time you need an instance with specific properties you call one of such factory methods.

The second approach is to make your own subclasses for different types of customisation:
```swift
class BrandNavigationController: UINavigationController {
  override func viewDidLoad() {
    super.viewDidLoad()
    // your customisation here
  }
}

let navigationController1 = BrandNavigationController(rootViewController: rootViewController)
```

You can also put the code into an extension:
```swift
extension UINavigationController {
  func customiseForPromoStyle() {
    // your customisation here
  }
}

let navigationController = UINavigationController(rootViewController: rootViewController)
navigationController.customiseForPromoStyle()
```

All these methods are valid options. But there is still one thing which cannot be incapsulated so easily. Bar button items are not properties of UINavigationController hence they have to be customised per each content view controller separately:

```swift
// inside the content view controller

navigationItem.backBarButtonItem = UIBarButtonItem(title: "", style: .plain, target: nil, action: nil)
let rightItem = UIBarButtonItem(title: "DoSmth", style: .done, target: nil, action: nil)
rightItem.setTitleTextAttributes(ViewController.regularBarButtonTextAttributes, for: .normal)
rightItem.setTitleTextAttributes(ViewController.regularBarButtonTextAttributes, for: .highlighted)
navigationItem.rightBarButtonItem = rightItem
```

Even if you move this code to some global styling function you still need to call it from the view controller's `viewDidLoad()` every time.

### UIAppearance

Most of you know about [UIAppearance proxy](https://developer.apple.com/documentation/uikit/uiappearance). It allows us to redefine the "default values" of the properties for specific UI-related classes. When we call `appearance()` method on a class we create a proxy object (which is a singleton for the class). Now all the instances of this class will be "styled" through this proxy right before being added to the view hierarchy if no other customisation is provided.

To style our navigation bars (all the navigation bars in the app) via UIAppearance we should write the following code:

```swift
UINavigationBar.appearance().barTintColor = UIColor.white
UINavigationBar.appearance().tintColor = UIColor.warm
UINavigationBar.appearance().isTranslucent = false
UINavigationBar.appearance().titleTextAttributes = ViewController.titleTextAttributes
UINavigationBar.appearance().shadowImage = UIImage()

UIBarButtonItem.appearance().setTitleTextAttributes(ViewController.regularBarButtonTextAttributes, for: .normal)
UIBarButtonItem.appearance().setTitleTextAttributes(ViewController.regularBarButtonTextAttributes, for: .highlighted)
```

With this set we have almost all the customisation in one place and applicable for all the instances of UINavigationBar as well as UIBarButtonItem. But there are some hidden pitfalls.
- UIAppearance cannot help us to customise back navigation buttons (for UIAppearance they are just regular UIBarButtonItems)
- Also keep in mind that UIAppearance proxy is basically a singleton (per class). And like every singleton it makes sense only when you don't need to change it during the app's lifecycle. Of course you can change the appearance whenever you want in the app but it's quite an error prone approach.
- There might be some difficulties when you need to add some variations into your styling scheme. Maybe you designed some exceptional UI elements for some exceptional cases. Or you have several kinds of user stories which are completely different UI-wise. Or you decided to soothly change the styling of the app from one to another by gradually updating ui-elements one after another spread between several app releases. In most cases you just define some "default" styling in UIAppearance and change it for the exceptional elements. But generally you lose some dynamism and flexibility and make your code more error prone.
- Customising through UIAppearance is implicit, especially if some other UI-customisation is done on the other levels (per UI instance, in subclasses or in nib/storyboards)
- If you have UIAppearance set for a navigation bar you cannot use the new UINavigationBarAppearance (more about that further)

## The dualism of UIBarButtonItems

Let's take a precise look at UIBarButtonItem. Can you quickly answer if it's a view or a model object?

UINavigationBar is definitely a view object. It inherits UIView and it's being rendered on screen. Navigation bar's properties describe HOW the content should be displayed: fonts, colours, elements positions, etc. WHAT to put into a navigation bar is a prerogative of a content view controller. Navigation controller is just a container and navigation bar's content changes according to a top view controller in the navigation stack. Specifically `navigationItem` is the view controller's property which contains the model for a navigation bar. It contains the title (which by default reflects the view controller's `title`), large title mode, prompt message, right and left bar buttons and a back button. From this point of view **bar button item is a model object**.

So as parts of view controller's `navigationItem` bar buttons act like a model. The class name **button item** implies that the actual UI element is a button and what we have here is just its item. The button is being handled completely by the navigation bar without us having access to it. But the thing is that besides the model (the button's title, image or a system button type) we also may provide some UI properties like colour and font for the text button. Actually if we want to visually customise the buttons that's our only way. The problem here is not only in blurring the architectural borders between the view and the model. The actual issue is that navigation bar - the UI-object - is responsible for styling all its subviews but the bar buttons.

When you want to style the navigation title, the background or the colour of the icons you change some properties of the navigation bar itself. This way it changes it for all the content view controllers in navigation stack. But if you have text bar buttons and you need to customise their appearance (colour, font, alignment) you cannot do it by setting some navigation bar's property. You have to change it for every bar button of every content view controller in the stack.

Using UIAppearance is one way to style bar buttons for all the view controllers at once. UIAppearance is basically a mechanism for styling UI objects. If you check [the documentation](https://developer.apple.com/documentation/uikit/uiappearance) (look for `Conforming Types`) you may find out that UIBarButtonItem (or actually it's parent class `UIBarItem`) is an exceptional case. It's the only class not inherited from UIView which conforms UIAppearance out of the box. Some of its properties which describe the look of the object can be customised through this proxy for the entire app. From this point of view **bar button item is a view object**.

That consists the view-model dualism of UIBarButtonItem.

### UINavigationBarAppearance

Apparently apple engineers knew about this pain so in XCode 11/iOS 13 the new approach to this problem was released. That's `UINavigationBarAppearance`.

UINavigationBarAppearance is a tree of configurations for the navigation bar. Starting from the root object (bar customisation) you go to leafs (title, bar buttons, back buttons) defining how different parts of the bar should be styled. You can even define how everything should behave for several different navigation bar modes: regular, compact (smaller navigation bar in a landscape mode) and when scrolled.

![](/images-posts/2019-11-06-navigation-bar-customisation-2/appearance-diagrame.png)

The system doesn't force you to fill in this graph of UI-configurations. There are several preset methods:
  - `configureWithDefaultBackground()`
  - `configureWithOpaqueBackground()`
  - `configureWithTransparentBackground()`

Using one of this functions you get all the configurations (bar, title, shadow, bar buttons,...) set by default with the more appropriate values. Then you just need to adjust some of them according to your needs.

One of the biggest impact from UINavigationBarAppearance is the new way of handling bar buttons. Now you can customise them per navigation bar completely removing any styling from the content view controller. Also you have separate way to customise the back button which also comes in quite handy.

```swift
let appearance = UINavigationBarAppearance()
appearance.configureWithOpaqueBackground()
appearance.titleTextAttributes = yourTitleTextAttributes
appearance.buttonAppearance.normal.titleTextAttributes = yourRegularBarButtonTextAttributes
appearance.doneButtonAppearance.normal.titleTextAttributes = yourBoldBarButtonTextAttributes
appearance.backButtonAppearance.normal.titleTextAttributes = yourBackButtonTextTextAttributes
// ...
navigationBar.scrollEdgeAppearance = appearance
navigationBar.compactAppearance = appearance
navigationBar.standardAppearance = appearance
```

Apple gives us several options with predefined values depending on the basic bar style: default, opaque, transparent. Other than this presets it also gives us adaptive colours out of the box.

Of course it doesn't deprive us from changing some UI settings for a specific view controller. If your view controller requires some specific appearance it can have its own `UINavigationBarAppearance`-instances which will be used to customise the navigation bar when this view controller is on screen:

```swift
let viewControllerAppearance = UINavigationBarAppearance()
// customising your appearance
viewController.navigationItem.standardAppearance = viewControllerAppearance
viewController.navigationItem.compactAppearance = viewControllerAppearance
```

Now finally we have UI customisation completely separated from the data model for the navigation bar. More over this UI customisation (an `appearance`) is not a global singleton but a property of specific object.

`UINavigationBarAppearance` is not the only one of its kind. It inherits all the general bar customisation functionality from its parent `UIBarAppearance` and shares it with the siblings: `UIToolbarAppearance` and `UITabBarAppearance`. These specific subclasses for toolbar and tabbar of course have their own specific properties but they all are specific applications of this new paradigm.

### Different approaches in one project

It's totally ok to use just UIAppearance proxy if all UI elements in the app have the same styling. Or if you want you can just have some customisation factories or set different styling in UINavigationController/UINavigationBar subclasses. Or if you are lucky enough and don't need to maintain iOS versions less than iOS 13 so just dive in to UINavigationBarAppearance and forget all the troubles. But in most cases you cannot just drop the old APIs right now and switch entirely to UINavigationBarAppearance.

If you use several approaches (or even several orderings of UIAppearance) in one project you should understand how the system resolve clashes when they happen. Meaning if two or more different customisation approaches have contradictive guidelines for the same object which one wins.

In [UIAppearance documentation](https://developer.apple.com/documentation/uikit/uiappearance) there is such a paragraph:
> In any given view hierarchy, the outermost appearance proxy wins. Specificity (depth of the chain) is the tie-breaker. In other words, the containment statement in `appearanceWhenContainedIn:` is treated as a partial ordering. Given a concrete ordering (actual subview hierarchy), UIKit selects the partial ordering that is the first unique match when reading the actual hierarchy from the window down.

Even after reading it several times I still don't get 100% of the message. But the general idea is that if you have some general proxy for let's say UIBarButtonItem:
```swift
UIBarButtonItem.appearance().setTitleTextAttributes(textAttributes1, for: .normal)
```
and some proxy specifically for the button contained in a navigation bar:
```swift
UIBarButtonItem.appearance(whenContainedInInstancesOf: [UINavigationBar.self]).setTitleTextAttributes(textAttributes2, for: .normal)
```
In case when the item sits in the navigation bar (so both settings are applicable) the system picks the most specific one.

If you have even more specific case for some trait collection:
```swift
UIBarButtonItem.appearance(for: someCollection, whenContainedInInstancesOf: [UINavigationBar.self]).setTitleTextAttributes(textAttributes3, for: .normal)
```
The last one with `textAttributes3` wins.

The same rule works when we mix UIAppearance with the instance customisation. Say we customise a specific instance of a bar button item inside the content view controller:
```swift
override func viewDidLoad() {
  let rightItem = UIBarButtonItem(barButtonSystemItem: .cancel, target: nil, action: nil)
  rightItem.setTitleTextAttributes(textAttributes4, for: .normal)
  navigationItem.rightBarButtonItem = rightItem
}
```
In this case regardless of all the settings in UIAppearance `textAttributes4` is applied, because that's the most specific case. The customisation of a specific instance overrides all the proxy (actually the proxy is not used at all in this case).

But what happens if we add new `UINavigationBarAppearance` to the same project?

Honestly when I first tried it we thought UINavigationBarAppearance for the specific bar will override all the general appearance proxies because it's more specific. But in practice we found out that every general proxy overrides UINavigationBarAppearance. So if you have some property customised by UINavigationBarAppearance for the specific navigation bar and the same property set in some global UIAppearance, UINavigationBarAppearance always loses.

So here is the ultimate chain of priority (from the least to the most):
- UINavigationBarAppearance
- class proxy - `appearance()`
- class proxy when contained in another class - `appearance(whenContainedInInstancesOf:)`
- class proxy when contained in another class with specific trait collection - `appearance(for:whenContainedInInstancesOf:)`
- instance customisation

Simply (though not technically correct), general UIAppearance overrides UINavigationBarAppearance, specific cases of UIAppearance override general UIAppearance, instance customisation overrides everything else.

As a result while we set some properties via UIAppearance in the project we cannot use new UINavigationBarAppearance for the same properties. It can be a serious issue for the big project when you cannot quickly define UINavigationBarAppearance-instances to all of your bars.

### P.S. Navigation bar shadow

That's just a poor victim of Apple's API changes. I already mentioned [in part 1](http://www.dmtopolog.com/navigation-bar-customization/) the differences in adjusting the shadow before/after iOS 11. In iOS 13 it changed again: not only new appearances were given to us, but also an opportunity to set just a colour to the shadow.

And just for fun: if you have code to switch on/off the shadow on your navigation bar and you support iOS versions 10, 11, 12 and 13 your code will look something like this:

```swift
public extension UINavigationBar {

    var shadowIsHidden: Bool {
        get {
            if #available(iOS 13.0, *) {
                return standardAppearance.shadowColor == nil
            } else if #available(iOS 11.0, *) {
                return shadowImage != nil
            } else {
                return shadowImage != nil && backgroundImage(for: .default) != nil
            }
        }

        set {
            guard shadowIsHidden != newValue else {
                return
            }

            let newShadowImage = newValue ? UIImage() : nil
            let newShadowColor = newValue ? nil : yourShadowColor
            if #available(iOS 13.0, *) {
                scrollEdgeAppearance?.shadowColor = newShadowColor
                compactAppearance?.shadowColor = newShadowColor
                standardAppearance.shadowColor = newShadowColor
            } else if #available(iOS 11.0, *) {
                shadowImage = newShadowImage
            } else {
                setBackgroundImage(newShadowImage, for: .default)
                shadowImage = newShadowImage
            }
        }
    }
}
```

&nbsp;

---
I hope you liked this piece of reading. If you have any questions, suggestions or corrections you can reach me out [on Twitter](https://twitter.com/dmtopolog)
