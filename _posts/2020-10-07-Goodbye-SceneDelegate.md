---

title: "Say Goodbye to SceneDelegate in SwiftUI"
date: 2020-10-07
categories: 

<!--comments: true-->

---

[image:A379283C-5436-459B-86ED-E21607C9D8B6-3816-00001649BB869DF5/0*jh0wml_bnqq3ipkq.jpeg]

[image:AC605DE2-7DEE-4DB6-9BF1-97C21F1BD30D-3816-00001649BB6969F5/_0*jh0wml_bnqq3ipkq.jpeg]
Photo by [Jonathan Kemper](https://unsplash.com/@jupp?utm_source=medium&amp;utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&amp;utm_medium=referral).

SwiftUI is a great way to build apps. It’s simple, concise, and fast. What’s made in UIKit can be recreated in SwiftUI with half the lines of code. What used to take weeks now only takes hours. But until today, it had a serious drawback: It depended on UIKit.

To display a view made with SwiftUI, you had to wrap it in a `UIHostingController`, which had to be wrapped in a `UIWindow`, which had to be defined in `SceneDelegate`:

For a framework designed for simplicity and performance… all that code doesn’t make sense. Why write 30 lines of code just to define a view?!

Today, at WWDC20, a solution was announced: `App`.

Huh, `App`? Yes, `App`. The `App`  protocol. It’s so simple it sounds weird after saying it a few times.

In Xcode 11.5 up to [Xcode 12 beta](https://developer.apple.com/download/), you’ll see a new option when creating a SwiftUI app: “SwiftUI App.”

What, is Apple trying to add an echo generator to Xcode? Not really — it’s actually one of the two options for “Life Cycle”:

[image:A638A4AB-8699-45B1-8BD9-96906C6EEF2A-3816-00001649BB403357/1*jFtnlCRtnqEL0KXrBnA1_A.png]

[image:EB9278F8-5360-435E-B612-8C44E731216C-3816-00001649BB18CAA2/1*fQUL9AnMvqHMggVkquqjgw.png]
“Life Cycle” refers to the application’s state — usually along the lines of “active,” “inactive,” or “background.” We use this to display the UI on launch as well as save user data on exit.

Previously, this was handled in `AppDelegate` and `SceneDelegate`, making it a hassle to manage application state — it was spread out across two files! Two. Files. Preposterous!

Well, if you select “SwiftUI App” and create the project, you’ll notice that both `AppDelegate` *and* `SceneDelegate` are gone! Instead, a file called `<App Name>App` replaces them!

[image:64AED02B-4DD9-4B31-8886-C7113E59EC1E-3816-00001649BAB79A32/1*xwIF4aV0xWzC7oR8NCLAag.png]
Wonder what’s in the `HelloWorldApp`? You won’t believe it:

That’s it! How? What magic did Apple do to replace two entire *files* of code with 17 lines (including comments)?

Although “Life Cycle” events sound important, most people will only touch `AppDelegate` or `SceneDelegate` once or twice. Code cleanup, state changes, and most other essential events are automatically handled. You don’t need to worry about any of it. And saving data? A good app saves on the fly — not at the end of its life cycle. The only part of the lifecycle that you actually need to manually modify is the entry point. Xcode needs to know what you want to display.

What Apple did with the `App` protocol was strip life cycles down to the single, most essential element — the entry point. Every additional life cycle is optional, and you can add it later.

Anyway, if you’re familiar with SwiftUI, our entry point’s syntax isn’t anything new. It’s declared inside a struct, making performance great while also making it easy to read.

Take the following code, for example, which renders some text that says “Hello World!”:

It looks very, very similar to our `HelloWorldApp` struct! There are some differences, however:

* `@main` tells Xcode that the following struct, `HelloWorldApp`, will be the entry point for the app. Only one struct can be marked with this attribute.
* According to [the documentation](https://developer.apple.com/documentation/swiftui/app), `App` is a protocol that “represents the structure and behavior of an app.” `HelloWorldApp` conforms to this. It’s like the base view of your app — no, the app itself. You’re literally writing out what your app will look like in this struct.
* `Scene` — The body of a SwiftUI `View` must be of type `View`. Similarly, the body of a SwiftUI `App` must be of type `Scene`…

Wait, what? Shouldn’t it be of type `App`? Well, actually, `Scene` makes more sense here:

> “Each scene contains the root view of a view hierarchy and has a life cycle managed by the system.” — [Apple Developer](https://developer.apple.com/documentation/swiftui/app)

The scene acts like a container for your views and is automatically managed for you:

> “The system decides when and how to present the view hierarchy in the user interface in a way that’s platform-appropriate and dependent on the current state of the app.” — [Apple Developer](https://developer.apple.com/documentation/swiftui/scene)

And because platforms like macOS and iPadOS support multiple windows, wrapping all your app’s views in a `Scene` makes reusability easier while also allowing for “Scene Phases” that include active, inactive, and background states.

* `WindowGroup` is a `Scene` that wraps views. The view that we want to present, `ContentView`, is a `View` — not a scene. `WindowGroup` lets us wrap them up into a single `Scene` that SwiftUI can recognize and display.

And that’s all the new syntax!

But what if you *need* to observe those optional life cycle events, like when the user exits the app? In the old `SceneDelegate`, it would look something like this:

That isn’t too bad, but there are separate functions for each life cycle event, making our code hard to manage.

In the new `App` protocol, Apple made sure that even the optional parts of the life cycle — those that not many will use — are still super easy to implement:

We first make a property, `scenePhase`, that gets the current state of activity from the system. The backslash ( `\` ) indicates that we’re using a [keypath](https://www.hackingwithswift.com/example-code/language/what-are-keypaths), which means we’re referring to the property itself and not its value. And whenever the property’s value changes, the `onChange` modifier is called, where we can get the life cycle state!

It’s that easy.

Will you ever go back to `SceneDelegate` and `AppDelegate`? If I had a choice, I wouldn’t.

The problem is that the new `App` protocol is only available for apps that use SwiftUI — those that use UIKit don’t get it. And SwiftUI still can’t compare to UIKit in terms of functionality — you don’t get collection views, coordinate-based animations, visual effect views, etc.

Also, sometimes you still need `AppDelegate` (for things like Firebase and home screen quick actions). In this case, you can continue using the `App` protocol — just extend it with `[@UIApplicationDelegateAdaptor](https://gist.github.com/aheze/6998b4bae660c8dade8ba6a75687772c)`.

At WWDC20 today, we got a lot of stuff we asked for, but we’re going to have to wait a couple more years for everything to arrive.

Thanks for reading! Happy coding and enjoy the rest of WWDC!

[Say Goodbye to SceneDelegate in SwiftUI](https://medium.com/better-programming/say-goodbye-to-scenedelegate-in-swiftui-444173b23015)