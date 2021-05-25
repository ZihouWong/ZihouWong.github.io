---

title: "区分 @State @StateObject @ObservedObject @EnvironmentObject "
date: 2020-11-18
categories: 

layout: post
<!--comments: true-->

---

## 区分 @State @StateObject @ObservedObject @EnvironmentObject 

### @State

> A property wrapper type that can read and write a value managed by SwiftUI

> It is designed to contain, simple value types, such as Ints, Strings, and Bools.

> It is not designed for more complex, reference types, such as classes or structs.

@State 是一个可以包装 "可读写的变量并由SwiftUI管理" 的属性包装器

适用于包装 “简单的，值类型”，比如 Int类型，String类型 以及 Bool类型的值

同理，不适用于包装“复杂的，引用类型”，比如 class类型，struct类型
 
#### 例子

```swift
// THIS IS GOOD
@State private var isPlaying = false

// THIS IS BAD
@State private var isPlaying = IsPlaying()
```

#### 测试：当@State 对象内的成员变量发生变化时，View是否会重新渲染

```swift
class TestObject {
	var num: Int = 0
}
```
```swift
struct StateTestView: View {
	@State var state = TestObject()
	
	var body: some View {
		VStack {
			Text("State: \(state.num)")
			Button("Increase state") {
				state.num += 1
				print("State: \(state.num)")
			}
		}
		.onChange(of: state.num) { newState in
			print("State: \(newState)")
		}
	}
}
```

#### 结果：对象内部的成员变量发生变化时，View不会重新渲染

> @State works by re-computing the body variable of your view any time it updates.

在任意时刻，被@State包装的变量 一旦发生变化，那么 View 将会被重新计算

But, because we’re using a complex, reference type, the value of state itself never changes. While a property of state , num has changed, the @State property wrapper has no idea because it is only watching the variable state , not any of its properties. To SwiftUI, because it is only watching state , it has no idea that num has changed, and so never re-renders the view.

但是因为我们在测试中 使用的是复杂的，引用类型（class），对象state并没有发生变化。尽管 对象state的成员变量 num 发生了变化，但是 @State属性包装器 并不能理解这次的变化，因为 @State属性包装器 只关注 state 对象，并不包括这个对象内部的成员变量。用SwiftUI的话来说，@State属性包装器 只关注 被包装的对象，不会因为 被包装的对象的成员变量 发生变化，而去重新绘制这个View。

This causes the scenario where we can see that the value is updating successfully in the console, but the body variable in our view is never re-computed and so we’re not seeing our UI update. This also causes our onChange method to never be called, as it detects changes between renders of the body variable, and if it is never rendered it is never called.

这就造成了...

So, as we can see, @State is not the right solution for anything more than the most simple piece of data. Think uses like keeping track if an element should be visible on the screen, or for highlighting a particular row number. For anything more complex, or related to your business logic, read on…

因此...

### 小结：

* @state object 只关注对象本身是否发生变化 比如是否为同一个对象，不关心 对象内部的成员变量是否发生变化

***

### @StateObject

> A property wrapper type that **instantiates** an observable object.

@StateObject 是一个可以**实例化** "@ObserableObject" 对象的属性包装器

什么是 ObservableObject?

> A type of object with a publisher that emits **before** the object has changed.

#### 测试： 当@StateObject 对象内的成员变量发生变化时，View是否会重新渲染

```swift
class TestObject: ObservableObject {
	@Published var num: Int = 0
}
```
```swift
struct StateObjectTestView: View {
	@StateObject var stateObject = TestObject()
	
	var body: some View {
		VStack {
			Text("State Object: \(stateObject.num)")
			Button("Increase state object") {
				stateObject.num += 1
				print("State object: \(stateObject.num)")
			}
		}
		.onChange(of: stateObject.num) { newStateObject in 
			print("State: \(newStateObject)")
		}
	}
}
```
当我们点击按钮的时候，我们不仅能够看到新的num值发生变化，并且我们还看到了UI也在随着值的变化而更新。

#### 结果：对象内部的成员变量发生变化时，View会重新渲染

This is because, by using @StateObject, we are letting our view know that whenever any of the @Published properties within the Observable Object change, we want the view to re-render. In other words, whenever we update num , because that property is @Published, it tells any views who are listening to it (observing it) that the value has changed and it should re-compute its views.

这是因为，使用@StateObject属性包装器之后，我们的View就会知道，只要@ObservableObject对象里的@Published 成员变量 发生变化，View就需要更新。换句话来说，当我们更新num的时候，因为使用了@Published 属性包装器，所以他会告诉所有正在监听这个变量的View，告诉他们这个变量num已经发生了变化，需要重新计算View

We don’t face the same issue as using @State, because now it is not just simply watching the object itself, but rather listening for changes to any of its marked @Published properties. Pretty cool.

我们现在面对的问题并不和使用@State的问题一样，因为像现在并不是简单的监视这个对象本身，而是监听这个对象中 所有标记了@Published 属性

The other important thing to note with @StateObject is the lifecycle of the object is directly tied to the view. By that, I mean if you enter into your view via a NavigationLink, set the num to 3, go back in the NavigationView and then re-enter your view again, that num will be reset to 0. In fact, the whole TestObject will have been created from scratch.

另外一个很重要的东西需要记住的是，使用@StateObject 会直接将对象的生命周期和View 相连，也就是说，如果你通过NavigationLink 进入你的View，并且在这个View中设置了num为3，当你退出这个View的时候，并且重新进入这个View，这个num会重新设置为0。事实上，是因为整个TestObject会重新实例化。

While it sounds annoying that your data is reset, this is actually the behaviour that we want. If we need data within a parent view (that is, the view that moves to your view with a NavigationLink), we should define the @StateObject in the parent view, not the child. If we only need the data in the child, then define the @StateObject in the child.

数据的重置确实会让人感到困惑，但是这就是我们想要实现的效果。如果我们需要在父View中使用在子View创建的数据，那么我们就应该在父View里通过@StateObject 修饰、创建对象，来进行数据的保存，而不是在子View中创建对象。如果我们只需要在子View中使用这些数据，那么使用@StateObject 修饰对象就够了。

Using this approach we can guarantee the data will not change or be disposed of if the view is still active. As Apple says in their documentation, this is the correct way to instantiate complex data types.
Our next two methods are to do with consuming complex objects that we have instantiated as a @StateObject.

使用这种方法，我们可以保证如果视图仍然处于活动状态，则数据不会更改或被丢弃。如Apple在其文档中所说，这是实例化复杂数据类型的正确方法。**接下来的两种方法与使用我们实例化为@StateObject的复杂对象有关。**

### 小结 

* 确认了ObservableObject 协议的类 需要有 @Published 的成员变量

* View 会关注 @StateObject 修饰的对象 中的 @Published 成员变量，一旦该变量发生变化，那么View也会重新渲染

* 如果需要传递@StateObject 对象的数据，就需要在父View中进行实例化。

***

###  @ObservedObject

Apple describes an @ObservedObject as:

> A property wrapper type that subscribes to an observable object and invalidates a view whenever the observable object changes.

这个属性包装器可以订阅一个Observable Object 并且可以当这个Observable Object 发生变化的时候，使View无效。

This is almost the same as an @StateObject, except it makes no mention of instantiation, or creation, of your variable.

这个和@StateObject 是基本相同的，除了没有提到变量的实例化或创建，仅仅只有 "Subscribe".

That’s because @ObservedObject is used to keep track of an object that has already been created, probably using @StateObject.@ObservedObject i

这是因为 @ObservedObject 常常用于追踪一个已经通过 @StateObject创建 的对象

@ObservedObject is to be used when you want to pass an Observable Object from one view to another view. You have already instantiated the Observable Object using @StateObject in the parent view, and now you want a child view to have access to the data. But, you don’t want to recreate the object again. This won’t retain any of the data within the object.

@ObservedObject 常常用于 当你想把一个 ObservableObjct 从一个View 传递到另外一个View的时候。你希望 子View 能够访问 一个 ObservableObject 对象 ，因为你已经在 父View中通过 @StateObject 实例化了这个对象，所以这时候你就不需要重新创建这个对象。即便你重新创建了这个对象，这个对象里也是没有保留任何数据的。

#### 传递@ObservedObject的代码：

```swift
// parent View
struct ContentView: View {
	@StateObject var observaedObject: TestObject = TestObject()
	var body: some View {
		TestView(observedObject: observedObject)
	}
}
```

```swift
// child View 
struct TestView: View {
	@ObservedObject var observedObject: TestObject
	
	var body: some View {
		Text("ObservedObject: \(observedObject.num)")
		Button("Increase observedObject") {
			observedObject.num += 1
			print("ObservedObject \(observedObject.num)")
		}
	}
}
```

As you can see, ChildView has one property called observedObject which accepts an object of type TestObject . It is marked as ObservedObject because it isn't instantiating it (that would be for @StateObject). Instead, it has already been instantiated and ChildView can now read and write to the same data that the parent view is reading and writing to.

如你所见，子View中有一个名为 observedObject 的属性，他接收一个类型为 TestObject 的对象。把这个属性用 ObservedObject 包装，是因为他不会自己实例化。相反的，这个属性是接受一个已经在父View中实例化的对象，现在 子View 和 父View 能够 **同时**对**同**一个数据进行读写

However, this approach can get quite cumbersome if many children views need to access the same data. For example, you might want to know the username of a user in many different views across your app, but you don’t want to have to pass down a UserDetails model to every single view.

尽管如此，如果需要将同一个数据从很多的View中传递，那么这将会很麻烦。比如说，你想在一个子View中得到用户的用户名数据，而在这子View之上 你需要穿过着许多的View才能到达App层，你并不希望App层一定要往下传递你的UserDetails Model 经过每一个View。

#### 小结

* @ObservedObject 不能用于实例化对象

* 需要搭配@StateObject使用，从父View实例化对象，通过@ObservedObject 传递到子View

*** 

Enter, @EnvironmentObject…

那么这时候就是需要 @EnvironmentObject了

@EnvironmentObject is for those scenarios where you need to use an ObservableObject but the views aren’t direct parent/child pairs. You may want to use a piece of data on the home view, and also deep within a settings menu, but you don’t want (or need) every view in between to know about that data — that would make for some messy code.

@EnvironmentObject 是为了那些
当你在View中需要 ObservableObject 对象的数据，但是这个View与实例化对象的 View 没有直接关系的时，就可以用到 @EnvironmentObject。你可能会想到通过一些臃肿的方法，从根View往下传递到叶子View，但是你并不希望这中间的View知道这传递对象的数据，因为这可能会写出一些混乱的代码

This is Apple’s recommended solution, as seen here from this WWDC video screenshot

![Apple’s recommended approach for data required in many disparate views. Source: Data Essentials in SwiftUI — WWDC 2020](https://miro.medium.com/max/1400/1*73FXsONzaMqz_i4cfrRnXg.png)
*Apple’s recommended approach for data required in many disparate views. Source: Data Essentials in SwiftUI — WWDC 2020*

Their ObservableObject has been created in the very first view, and they want some views much further down to know and respond, to when it is changed.

ObservableObject 对象已经在最开始View创建了，接着他们希望在分支下的某些View同样可以访问这个ObservableObject 对象的数据，并且当这个数据发生变化的时候，这些View将重新计算

There are two parts to using @EnvironmentObject.

当使用EnvironmentObject的时候，有两个重要的点需要记住

Firstly, you need to create an object to use. After you create it, you then need to attach it to a view for all child views to have access to it. Let’s go ahead and do that.

首先，这个EnvironmentObject需要你先创建才能使用，当你创建完这个对象之后，你需要将这个对象附在包含所有需要用到数据的子View的 根View上。

```swift
@main
struct TestApp: App {	
	@StateObject var environmentObject = TestObject()
	var body: some View {
		WindowGroup {
			ContentView().environmentObject(environmentObject)
		}	
	}
}
```

Here, on line 3 I’ve created the object, and then on line 6, I’ve attached it to my view. Notice I’ve done this all in my main app file. This is a fairly common pattern for data you want accessible across your entire app.

这里在第三行创建了这个对象，然后在第6行将这对附在了根View上。对于要在整个应用程序中访问的数据，这是一种相当常见的模式。

Then, to use the object in any view that is within ContentView , you do this:

然后在ContentView中的任意子View中都可以这样使用这个对象

```swift
@EnvironmentObject var environmentObject: TestObject
```

This is exactly like you would use @ObservedObject, except this time we are expecting it to be in the environment, as opposed to directly passed from the parent.

这和使用 @observedObject 完全一样，特别这次我们把这个对象存放在 全局中，并且可以从根直接获取这个数据

The way @EnvironmentObject works is when called within a view, it looks from an object of that type in the environment (in other words, from any parent above it that has specified an environmentObject), and then lets you use it. It does this at run time, as opposed to at compile-time, so if you haven’t set up your environment object properly, your app will crash when it goes to use it.

@EnvironmentObject 的工作方式是，当在子View中你调用这个对象时，系统会从这个环境中查找这个对象。换句话说，父View需要先实例化这个EnvironmentObject了，子View才能使用这个对象。在子View中调用这个对象是在runtime，而不是compile time。因此如果在子View中调用这个对象的时候，要是这个对象没有正确设置好，那么程序就将崩溃。

The other point to note is @EnvironmentObject looks for an object of that type. This means you can’t define more than one environment object in the same view tree with the same type.

另外一个重点是，@EnvironmentObject 查找一个类型的对象，这就意味着，在同一个视图树中你不能使用同一个类型去定义多个对象。

#### 小结

* 同一个视图树中，有且只有一个相同类型的对象

* 在使用这个对象之前，必须确保这个对象的正确无误，否则会导致崩溃

*** 
