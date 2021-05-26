---
title: "Swift中的高阶函数"
date: 2020-10-08
categories: 


<!--comments: true-->

---


  

* 首页

	* [首页](https://juejin.im/)
	* [沸点](https://juejin.im/pins)
	* [话题](https://juejin.im/topics)
	* [小册](https://juejin.im/books)
	* [活动](https://juejin.im/events/all)

* 写文章
	* 发布沸点

* 登录

  [_GodIsCoder](https://juejin.im/user/3280598426209015)   2019年06月05日  阅读 1556 

关注
# Swift中的高阶函数: Filter, Map, Reduce, flatmap, compactMap(译)

自从我详细了解函数和闭包后，我就想去知道它们的优点和在编程中的使用， 我的理解是高阶函数的使用是基于集合类型的。

根据我的理解，高阶函数就是把另一个函数或者闭包当做参数并且返回值。

首先，我来解释一下。思考下面的代码将会使你对高阶函数的理解更深：

```
// 将函数当做参数传递给另一个函数

func addation(num1: Double, num2: Double) -> Double {
    return num1 + num2
}

func multiply(num1: Double, num2: Double) -> Double {
    return num1 * num2
}

func doMathOperation(operation: (_ x: Double, _ y: Double) -> Double, num1: Double, num2: Double) -> Double {
    return operation(num1, num2)
}

// print 12
doMathOperation(operation: addation(num1:num2:), num1: 10, num2: 2)
//print 20
doMathOperation(operation: multiply(num1:num2:), num1: 10, num2: 2)
复制代码
```

`addation(num1:num2:)`、`multiply(num1:num2:)` 都是 `(Double,Double)->Double` 的函数。 `addation(num1:num2:)` 接受两个 Double 的值，并返回它们的和； `multiply(num1:num2:)` 接受两个 Double 的值，并返回它们的乘积。 `doMathOperation(operation:num1:num2:)` 则是一个接受三个参数的高阶函数，它接受两个 Double 的参数和一个 `(Double,Double)->Double` 类型的函数。看一下该函数的调用，你应该就理解了高阶函数的工作原理。

在Swift中，函数和闭包是 `一等公民` 。它们可以存储在变量中被传递。

```
// 函数返回值为另一个函数
func doArithmeticOperation(isMultiply: Bool) -> (Double, Double) -> Double {
    func addation(num1: Double, num2: Double) -> Double {
        return num1 + num2
    }
    
    func multiply(num1: Double, num2: Double) -> Double {
        return num1 * num2
    }
    
    return isMultiply ? multiply : addation
}

let operationToPerform1 = doArithmeticOperation(isMultiply: true)
let operationToPerform2 = doArithmeticOperation(isMultiply: false)

operationToPerform1(10, 2) //20
operationToPerform2(10, 2) //12
复制代码
```

在上面的代码中， `doArithmeticOperation(isMultiply:)` 是一个返回值类型为 `(Double,Double)->Double` 的高阶函数。它基于一个布尔值来判断返回值。operationToPerform1 执行乘的操作，operationToPerform2 执行加的操作。通过上面的函数定义和调用，你就理解所有的事情了。

当然，你可以通过多种不同的方法来实现一样的东西。你可以使用闭包来代替函数，你可以使用枚举来判断函数的操作。在这里，我只是通过上面的代码来解释一下什么是高阶函数。

下面是 Swift 库中自带的一些高阶函数，如果我理解没错的话，下面的函数接受闭包类型的参数。你可以在集合类型(Array Set Dictionary)中使用它们。如果你对闭包不太了解，你可以通过 [这篇文章](https://medium.com/@abhimuralidharan/functional-swift-all-about-closures-310bc8af31dd) 来学习。

### Map

**map 函数的作用就是对集合进行一个循环，循环内部再对每个元素做同一个操作。** 它返回一个包含映射后元素的数组。

#### Map on Array：

假设我们有一个整数型数组：

```
let arrayOfInt = [2,3,4,5,4,7,2]
复制代码
```

我们如何让每个数都乘10呢？我们通常使用 `for-in` 来遍历每个元素，然后执行相关操作。

```
var newArr: [Int] = []
for value in arrayOfInt { newArr.append(value*10) }
print(newArr) // prints [20, 30, 40, 50, 40, 70, 20]
复制代码
```

上面的代码看着有点冗余。它包含创建一个新数组的样板代码，我们可以通过使用 map 来避免。我们通过 Swift 的自动补全功能可以看到 map 函数接受有一个 Int 类型的参数并返回一个泛型的闭包。

[image:6F0CBD12-1E81-4323-81A9-CC29B642638F-4728-00000CD53FF627C5/16b208da0b5297d3.jpeg]

```
let newArrUsingMap = arrayOfInt.map { $0 * 10 }
复制代码
```

对于一个整型数组，这是 map 的极简版本。我们可以在闭包中使用 `$` 操作符来代指遍历的每个元素。

下面的代码作用都是一样的，它们表示了一个从繁到简的过程。通过下面的代码，你应该对闭包有了一个清晰的认识。

[image:79D5993C-FF45-4819-949D-18A2CC81A6C2-4728-00000CD53F983741/16b20912ab040c6e.jpeg]
**map 的工作原理** ：map 函数接受一个闭包作为参数，在迭代集合时调用该闭包。这个闭包映射集合中的元素，并将结果返回。map 函数再将结果放在数组中返回。

#### Map on Dictionary：

假设我们有一个书名当做 key ，书的价格当做 value 的字典。

```
let bookAmount = [“harrypotter”:100.0, “junglebook”:100.0]
复制代码
```

如果你试图 map 这个字典，Swift 的自动补全将是这样：

[image:A3ECAE20-5703-4B0A-A143-1BD2A97770EE-4728-00000CD53F6C41FB/16b255140a22e344.jpeg]

```
let bookAmount = ["harrypotter": 100.0, "junglebook": 100.0]
let returnFormatMap = bookAmount.map { (key, value) in
    return key.capitalized
}
print(returnFormatMap)
//["Junglebook", "Harrypotter"]
复制代码
```

我们通过上面的代码，对一个字典进行遍历，每次遍历在闭包中都有一个 String 类型的 key ，和一个 Double 类型的 value 。返回值为一个大写首字母的字符串数组，数组的值还可以是价格或者元组，这取决于你的需求。

`注意：map 函数的返回值类型总是一个泛型数组。你可以返回包含任意类型的数组。`

#### Map on set:

```
let lengthInMeters: Set = [4.0, 6.2, 8.9]
let lengthInFeet = lengthInMeters.map { $0 * 3.2808 }
print(lengthInFeet)
//[20.340960000000003, 13.1232, 29.199120000000004]
复制代码
```

在上面的代码中，我们有一个值类型为 Double 的 set，我们的闭包返回值也是 Double 类型。 `lengthInMeters` 是一个 set，而 `lengthInFeet` 是一个数组。

#### 如果你想在 map 的时候获取 index 应该怎么做？

答案很简单，你必须在 map 之前调用 `enumerate` 。

[image:7869CEB6-BA8F-4248-B1D8-D1EC26BDA1BC-4728-00000CD53EBC2FE6/16b255fbf97ae77c.jpeg]
">

下面是示例代码：

```
let numbers = [1, 2, 4, 5]
let indexAndNum = numbers.enumerated().map { (index,element) in
return "\(index):\(element)"
}
print(indexAndNum) // [“0:1”, “1:2”, “2:4”, “3:5”]
复制代码
```

### Filter

`filter` 会遍历集合，返回一个包含符合条件元素的集合。

#### Filter on array

假设我们要筛选一个整型数组中包含的偶数，你可能会写下面的代码：

```
let arrayOfIntegers = [1, 2, 3, 4, 5, 6, 7, 8, 9]

var newArray = [Int]()

for integer in arrayOfIntegers {
    if integer % 2 == 0 {
        newArray.append(integer)
    }
}

print(newArray) //[2, 4, 6, 8]
复制代码
```

就像 map ，这是一个简单的函数去筛选集合中的元素。

如果我们对整形数组使用 filter ，Swift 的自动补全展示如下：

[image:B787774E-9E68-422A-9E66-5950E7046307-4728-00000CD53E964B70/16b2569a261ae6fc.jpeg]
">

如你所见， filter 函数调用了一个接受一个 Int 类型参数、返回值为 Bool 类型、名字为 `isIncluded` 的闭包。 `isIncluded` 在每次遍历都会返回一个布尔值，然后基于布尔值将创建一个新的包含筛选结果的数组。

我们可以通过 filter 将上面的代码修改为：

```
var newArray = arrayOfIntegers.filter { (value) -> Bool in
    return value % 2 == 0
}
复制代码
```

filter 闭包也可以被简化，就像 map：

[image:6E621D43-27E0-4EAE-A7F3-562DAD86A413-4728-00000CD53DDDF80E/16b256f340acc0f1.jpeg]
">

#### Filter on dictionary

假设有一个书名当做 key ，书的价格当做 value 的字典。

```
let bookAmount = ["harrypotter":100.0, "junglebook":1000.0]
复制代码
```

如果你想对这个字典调用 filter 函数，Swift 的自动补全将是这样：

[image:8165ECBF-B6D4-4716-BC4F-685BF6B3CC2C-4728-00000CD53DB14872/16b25718b662bc53.jpeg]
">

filter 函数会调用一个名字为 isIncluded 的闭包，该闭包接受一个键值对作为参数，并返回一个布尔值。最终，基于返回的布尔值，filter 函数将决定是否将键值对添加到数组中。

`<重要> 原文中作者写道：对字典调用 Filter 函数，将返回一个包含元组类型的数组。但译者在 playground 中发现 返回值实际为字典类型的数组。`

```
let bookAmount = ["harrypotter": 100.0, "junglebook": 1000.0]
let results = bookAmount.filter { (key, value) -> Bool in
    return value > 100
}

print(results) //["junglebook": 1000.0]
复制代码
```

[image:5C6D4E18-9807-42CD-B302-DD5EFB41A2ED-4728-00000CD53D860B4F/16b257814370c7b5.jpeg]
">

还可以将上述代码简化：

```
//$0 为 key $1 为 value
let results = bookAmount.filter { $1 > 100 }
复制代码
```

#### Filter on set

```
let lengthInMeters: Set = [1, 2, 3, 4, 5, 6, 7, 8, 9]
let lengthInFeet = lengthInMeters.filter { $0 > 5 }
print(lengthInFeet) //[9, 8, 7, 6]
复制代码
```

在每次遍历时， filter 闭包接受一个 Double 的参数，返回一个布尔值。筛选数组中包含的元素基于返回的布尔值。

### Reduce

reduce ：联合集合中所有的值，并返回一个新值。

Apple 的官方文档如下：

```
func reduce<Result>(_ initialResult: Result, _ nextPartialResult: (Result, Element) throws -> Result) rethrows -> Result
复制代码
```

reduce 函数接受两个参数：

* 第一个为初始值，它用来存储初始值和每次迭代中的返回值。
* 另一个参数是一个闭包，闭包包含两个参数：初始值或者当前操作的结果、集合中的下一个 item 。

#### Reduce on arrays

让我们通过一个例子来理解 reduce 的具体作用：

```
let numbers = [1, 2, 3, 4]
let numberSum = numbers.reduce(0) { (x, y)  in
    return x + y
}
print(numberSum) // 10
复制代码
```

reduce 函数会迭代4次。

```
1.初始值为0，x为0，y为1 -> 返回 x + y 。所以初始值或者结果变为 1。
2.初始值或者结果变为 1，x为1，y为2 -> 返回 x + y 。所以初始值或者结果变为 3。
3.初始值或者结果变为 3，x为3，y为3 -> 返回 x + y 。所以初始值或者结果变为 6。
4.初始值或者结果变为 6，x为6，y为4 -> 返回 x + y 。所以初始值或者结果变为 10。
复制代码
```

reduce 函数可以简化为：

```
let reducedNumberSum = numbers.reduce(0) { $0 + $1 }
print(reducedNumberSum) // prints 10
复制代码
```

[image:837ED1F5-466D-4867-A8F4-4EEB8161192A-4728-00000CD53CE864E7/16b259771af1183f.jpeg]
">

在本例中，闭包的类型为 `(Int,Int)->Int` 。所以，我们可以传递类型为 `(Int,Int)->Int` 的任意函数或者闭包。比如我们可以把操作符替换为 -, *, / 等。

```
let reducedNumberSum = numbers.reduce(0,+) // returns 10
复制代码
```

我们可以在闭包里添加 `*` 或者其他的操作符。

```
let reducedNumberSum = numbers.reduce(0) { $0 * $1 }
// reducedNumberSum is 0...
复制代码
```

上面的代码也可以写成这样：

```
let reducedNumberSum = numbers.reduce(0,*)
复制代码
```

reduce 也可以通过 + 操作符来合并字符串。

```
let codes = ["abc","def","ghi"]
let text = codes.reduce("") { $0 + $1} //the result is "abcdefghi"
or
let text = codes.reduce("",+) //the result is "abcdefghi"
复制代码
```

#### Reduce on dictionary

让我们来 reduce bookAmount。

```
let bookAmount = ["harrypotter":100.0, "junglebook":1000.0]

let reduce1 = bookAmount.reduce(10) { (result, tuple) in
    return result + tuple.value
}
print(reduce1) //1110.0

let reduce2 = bookAmount.reduce("book are ") { (result, tuple) in
    return result + tuple.key + " "
}
print(reduce2) //book are junglebook harrypotter 
复制代码
```

对于字典，reduce 的闭包接受两个参数。

```
1.一个应该被 reduce 的初始值或结果
2.一个当前键值对的元组
复制代码
```

reduce2 可以被简化为：

```
let reducedBookNamesOnDict = bookAmount.reduce("Books are ") { $0 + $1.key + " " } //or $0 + $1.0 + " "
复制代码
```

#### Reduce on set

Set 中的 reduce 使用和数组中的一致。

```
let lengthInMeters: Set = [4.0, 6.2, 8.9]
let reducedSet = lengthInMeters.reduce(0) { $0 + $1 }
print(reducedSet) //19.1

复制代码
```

闭包中的返回值类型为 Double。

### Flatmap

Flatmap 用来铺平 collections 中的 collection 。在铺平 collection 之前，我们对每一个元素进行 map 操作。 `Apple docs 解释: 返回一个对序列的每个元素进行形变的串级结果( Returns an array containing the concatenated results of calling the given transformation with each element of this sequence.)`

```
解读 : map + (Flat the collection)
复制代码
```

[image:7E0842C9-BB90-4849-A467-A62461324A16-4728-00000CD53C5FB67C/16b25ab438f3251e.jpeg]
"> `图1 对 flatmap 进行代码说明`

[image:837D1A0E-8F16-4B26-97BD-99D8672F318C-4728-00000CD53C4071E2/16b25ab6137d5395.jpeg]
"> `图2`

在图2 中，flatMap 迭代 collections 中的所有 collection 进行大写操作。在这个例子中，每个 collection 是字符串。下面是执行步骤：

1. 对所有的字符串执行 `upperCased()` 函数，这类似于:

```
[“abc”,”def”,”ghi”].map { $0.uppercased() }
复制代码
```

输出：

```
output: [“ABC”, “DEF”, “GHI”]
复制代码
```

1. 将 collections 铺平为一个 collection。

```
output: ["A", "B", "C", "D", "E", "F", "G", "H", "I"]
复制代码
```

[image:1D4EA904-CA79-4773-8DEB-85C88F50C10A-4728-00000CD53C161223/16b25b2c67bee6a9.jpeg]
">

`注意：在 Swift3 中，flatMap 还可以自动过滤 nil 值。但是现在已经废弃该功能。现在用 compactMap 来实现这一功能，稍后在文章中我们会讲到。`

[image:C75D6E9B-E4EF-49A6-8333-3F43CAB9DBDC-4728-00000CD53B653DD6/16b25b52b702b52b.jpeg]
">

现在，你应该明白了 flatMap 是做什么得了。

#### Flatmap on array

```
let arrs = [[1,2,3], [4, 5, 6]]
let flat1 = arrs.flatMap { return $0 }
print(flat1) //[1, 2, 3, 4, 5, 6]
复制代码
```

#### Flatmap on array of dictionaries

因为在铺平之后的返回数组包含元素的类型为元组。所以我们不得不转换为字典。

```
let arrs = [["key1": 0, "key2": 1], ["key3": 3, "key4": 4]]
let flat1 = arrs.flatMap { return $0 }
print(flat1)
//[(key: "key2", value: 1), (key: "key1", value: 0), (key: "key3", value: 3), (key: "key4", value: 4)]

var dict = [String: Int]()

flat1.forEach { (key, value) in
    dict[key] = value
}

print(dict)
//["key4": 4, "key2": 1, "key1": 0, "key3": 3]
复制代码
```

#### Flatmap on set

[image:62F809ED-7451-477D-8706-009EBFF0B495-4728-00000CD539D97E99/16b25bdb9c68ce0e.jpeg]
">

#### Flatmap by filtering or mapping

我们可以用 flatMap 来实现将一个二维数组铺平为一维数组。 flatMap 的闭包接受一个集合类型的参数，在闭包中我们还可以进行 filter map reduce 等操作。

```
let collections = [[5, 2, 7], [4, 8], [9, 1, 3]]
let onlyEven = collections.flatMap { (intArray) in
    intArray.filter({ $0 % 2 == 0})
}
print(onlyEven) //[2, 4, 8]
复制代码
```

上述代码的简化版：

```
let onlyEven = collections.flatMap { $0.filter { $0 % 2 == 0 } }
复制代码
```

### 链式 : (map + filter + reduce)

我们可以链式调用高阶函数。 `不要链接太多，不然执行效率会慢。下面的代码我在playground中就执行不了。`

```
let arrayOfArrays = [[1, 2, 3, 4], [5, 6, 7, 8, 4]]
let sumOfSquareOfEvenNums = arrayOfArrays.flatMap{$0}.filter{$0 % 2 == 0}.map{$0 * $0}.reduce {0, +}
print(sumOfSquareOfEvenNums) // 136

//这样可以运行
let SquareOfEvenNums = arrayOfArrays.flatMap{$0}.filter{$0 % 2 == 0}.map{$0 * $0}
let sum = SquareOfEvenNums.reduce(0 , +) // 136
复制代码
```

### CompactMap

在迭代完集合中的每个元素的映射操作后，返回一个非空的数组。

```
let arr = [1, nil, 3, 4, nil]
let result = arr.compactMap{ $0 }
print(result) //[1, 3, 4]
复制代码
```

它对于 Set ,和数组是一样的作用。

```
let nums: Set = [1, 2, nil]
let r1 = nums.compactMap { $0 }
print(r1) //[2, 1]
复制代码
```

而对于 Dictionary ，它是没有任何作用的，它只会返回一个元组类型的数组。所以我们需要使用 compactMapValues 函数。(该函数在Swift5发布)

```
let dict = ["key1": nil, "key2": 20]
let result = dict.compactMap{ $0 }
print(result) //[(key: "key1", value: nil), (key: "key2", value: Optional(20))]

let dict = ["key1": nil, "key2": 20]
let result = dict.compactMapValues{ $0 }
print(result)
复制代码
```

#### Tip

```
let arr = [1, nil, 3, 4, nil]
let result = arr.map { $0 }
print(result) //[Optional(1), nil, Optional(3), Optional(4), nil]
复制代码
```

使用 map 是这样的。

### 总结

好了，到这里这篇文章也就接近尾声了。

我们要尽可能的使用高阶函数：

* 它可以提高你的 Swift 技能
* 代码可读性更高
* 更加函数式

关注下面的标签，发现更多相似文章
[Swift](https://juejin.im/tag/Swift) [函数式编程](https://juejin.im/tag/%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BC%96%E7%A8%8B)

  [_GodIsCoder](https://juejin.im/user/3280598426209015)  iOS开发
[发布了 94 篇专栏 ·](https://juejin.im/user/3280598426209015/posts)  获得点赞 679 ·   获得阅读 78,344 

关注

[安装掘金浏览器插件](https://juejin.im/extension/?utm_source=juejin.im&amp;utm_medium=post&amp;utm_campaign=extension_promotion) 打开新标签页发现好内容，掘金、GitHub、Dribbble、ProductHunt 等站点内容轻松获取。快来安装掘金浏览器插件获取高质量内容吧！

 
[想说却还没说](https://juejin.im/user/1732486054549598)

"filter 会遍历集合，返回一个包含符合条件元素的数组。" filter返回的不是数组，应该是原来的集合类型

8月前  · 删除
1
回复

 
[_GodIsCoder](https://juejin.im/user/3280598426209015) (作者)
iOS开发

已修改

8月前  · 删除

回复

 

相关推荐
	* [老司机技术周报](https://juejin.im/user/1926000101569870)

	* 2天前
	* [Swift](https://juejin.im/tag/Swift) [Flutter](https://juejin.im/tag/Flutter)

[老司机 iOS 周报 #130 | 2020-09-28](https://juejin.im/post/6877746452706099214)
	* [7]()
	* [2](https://juejin.im/post/6877746452706099214#comment)
	* 微博
	* 微信扫一扫

	* [说过的玩笑](https://juejin.im/user/1714893871133079)

	* 3天前
	* [Swift](https://juejin.im/tag/Swift)

[迟到的Swift入门 - UITableView](https://juejin.im/post/6877434365996072973)
	* [3]()
	* 
	* 微博
	* 微信扫一扫

	* [老司机技术周报](https://juejin.im/user/1926000101569870)

	* 10天前
	* [Swift](https://juejin.im/tag/Swift) [Flutter](https://juejin.im/tag/Flutter)

[老司机 iOS 周报 #129 | 2020-09-21](https://juejin.im/post/6874818350313275405)
	* [10]()
	* [3](https://juejin.im/post/6874818350313275405#comment)
	* 微博
	* 微信扫一扫

	* [素数心情](https://juejin.im/user/2101921961221943)

	* 3天前
	* [Swift](https://juejin.im/tag/Swift)

[模块化学习实践总结](https://juejin.im/post/6877081524341538823)
	* 
	* 
	* 微博
	* 微信扫一扫

	* [老司机技术周报](https://juejin.im/user/1926000101569870)

	* 17天前
	* [Swift](https://juejin.im/tag/Swift) [Flutter](https://juejin.im/tag/Flutter)

[老司机 iOS 周报 #128 | 2020-09-14](https://juejin.im/post/6872233282096873486)
	* [9]()
	* 
	* 微博
	* 微信扫一扫

	* [孟思行](https://juejin.im/user/1187128287435517)

	* 7天前
	* [函数式编程](https://juejin.im/tag/%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BC%96%E7%A8%8B)

[轻松玩转函数式编程](https://juejin.im/post/6875840102220693518)
	* [14]()
	* [2](https://juejin.im/post/6875840102220693518#comment)
	* 微博
	* 微信扫一扫

	* [东方教主](https://juejin.im/user/61228381119038)

	* 9天前
	* [Swift](https://juejin.im/tag/Swift)

[低仿扫描全能王的选择区域功能](https://juejin.im/post/6875134095232335880)
	* [3]()
	* 
	* 微博
	* 微信扫一扫

	* [老司机技术周报](https://juejin.im/user/1926000101569870)

	* 24天前
	* [Swift](https://juejin.im/tag/Swift) [Flutter](https://juejin.im/tag/Flutter)

[老司机 iOS 周报 #127 | 2020-09-07](https://juejin.im/post/6869622502302154765)
	* [11]()
	* [2](https://juejin.im/post/6869622502302154765#comment)
	* 微博
	* 微信扫一扫

	* [十月_月读](https://juejin.im/user/4388906150667736)

	* 13天前
	* [Swift](https://juejin.im/tag/Swift)

[Swift 5.x - 内存安全（中文文档）](https://juejin.im/post/6873633524663091214)
	* [5]()
	* 
	* 微博
	* 微信扫一扫

	* [老司机技术周报](https://juejin.im/user/1926000101569870)

	* 1月前
	* [Swift](https://juejin.im/tag/Swift) [Flutter](https://juejin.im/tag/Flutter)

[老司机 iOS 周报 #125 | 2020-08-24](https://juejin.im/post/6864431285935276040)
	* [14]()
	* 
	* 微博
	* 微信扫一扫

	* [longitachi](https://juejin.im/user/4133773453304141)

	* 24天前
	* [Swift](https://juejin.im/tag/Swift)

[一款满足大多日常开发的照片选择框架](https://juejin.im/post/6869656157724606477)
	* [22]()
	* [10](https://juejin.im/post/6869656157724606477#comment)
	* 微博
	* 微信扫一扫

	* [说过的玩笑](https://juejin.im/user/1714893871133079)

	* 20天前
	* [Swift](https://juejin.im/tag/Swift)

[迟到的Swift入门 - 数组操作](https://juejin.im/post/6871047949250461710)
	* [5]()
	* [1](https://juejin.im/post/6871047949250461710#comment)
	* 微博
	* 微信扫一扫

	* [码农小胖哥](https://juejin.im/user/4107431172378887)

	* 13天前
	* [Java](https://juejin.im/tag/Java) [函数式编程](https://juejin.im/tag/%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BC%96%E7%A8%8B)

[Java中的::符号挺好理解的](https://juejin.im/post/6873479007703138317)
	* [8]()
	* 
	* 微博
	* 微信扫一扫

	* [小亲亲亲0](https://juejin.im/user/2612095356765614)

	* 13天前
	* [Swift](https://juejin.im/tag/Swift)

[重学Swift(一)漫谈一些基础❲更新中❳](https://juejin.im/post/6873429230382743566)
	* [1]()
	* [1](https://juejin.im/post/6873429230382743566#comment)
	* 微博
	* 微信扫一扫

	* [老司机技术周报](https://juejin.im/user/1926000101569870)

	* 1月前
	* [Swift](https://juejin.im/tag/Swift) [WWDC](https://juejin.im/tag/WWDC)

[老司机 iOS 周报 #126 | 2020-08-31](https://juejin.im/post/6867021422338768903)
	* [10]()
	* 
	* 微博
	* 微信扫一扫

	* [老司机技术周报](https://juejin.im/user/1926000101569870)

	* 1月前
	* [Swift](https://juejin.im/tag/Swift) [Flutter](https://juejin.im/tag/Flutter)

[老司机 iOS 周报 #123 | 2020-08-10](https://juejin.im/post/6859231285207171086)
	* [14]()
	* 
	* 微博
	* 微信扫一扫

	* [说过的玩笑](https://juejin.im/user/1714893871133079)

	* 1月前
	* [Swift](https://juejin.im/tag/Swift)

[迟到的Swift入门 - UIButton](https://juejin.im/post/6867394035363872781)
	* [4]()
	* 
	* 微博
	* 微信扫一扫

	* [Shift](https://juejin.im/user/2022711347126904)

	* 21天前
	* [Swift](https://juejin.im/tag/Swift)

[Swift 范型（Generics译文）](https://juejin.im/post/6870424765958455303)
	* [8]()
	* [2](https://juejin.im/post/6870424765958455303#comment)
	* 微博
	* 微信扫一扫

	* [yalewan](https://juejin.im/user/3280598427770424)

	* 14天前
	* [函数式编程](https://juejin.im/tag/%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BC%96%E7%A8%8B)

[函数式编程指南第一弹](https://juejin.im/post/6873276859744780301)
	* [11]()
	* [2](https://juejin.im/post/6873276859744780301#comment)
	* 微博
	* 微信扫一扫

	* [printf](https://juejin.im/user/2260251635890615)

	* 12天前
	* [Swift](https://juejin.im/tag/Swift)

[Swift：4个简化属性设置代码的神奇技巧 【译】](https://juejin.im/post/6873748346591756295)
	* [1]()
	* [4](https://juejin.im/post/6873748346591756295#comment)
	* 微博
	* 微信扫一扫

关于作者
[_GodIsCoder](https://juejin.im/user/3280598426209015) iOS开发

获得点赞679
文章被阅读78,344

* 
* 
* 
相关文章
[SnapKit入门教程  22  8](https://juejin.im/post/6844903615191056397) [iOS 头条一面 面试题  51  20](https://juejin.im/post/6844904063553765390) [如何用Swift 实现一个单链表？  2  0](https://juejin.im/post/6847009772680511495) [Swift 中的 as，as?，as! 的区别  18  9](https://juejin.im/post/6858039433603022862) [iOS调试进阶-更高效的使用Xcode和LLDB  39  8](https://juejin.im/post/6844903866345996296)

目录
* [Map](https://juejin.im/post/6844903860352319495#heading-0)

	* [Map on Array：](https://juejin.im/post/6844903860352319495#heading-1)
	* [Map on Dictionary：](https://juejin.im/post/6844903860352319495#heading-2)
	* [Map on set:](https://juejin.im/post/6844903860352319495#heading-3)
	* [如果你想在 map 的时候获取 index 应该怎么做？](https://juejin.im/post/6844903860352319495#heading-4)

* [Filter](https://juejin.im/post/6844903860352319495#heading-5)

	* [Filter on array](https://juejin.im/post/6844903860352319495#heading-6)
	* [Filter on dictionary](https://juejin.im/post/6844903860352319495#heading-7)
	* [Filter on set](https://juejin.im/post/6844903860352319495#heading-8)

* [Reduce](https://juejin.im/post/6844903860352319495#heading-9)

	* [Reduce on arrays](https://juejin.im/post/6844903860352319495#heading-10)
	* [Reduce on dictionary](https://juejin.im/post/6844903860352319495#heading-11)
	* [Reduce on set](https://juejin.im/post/6844903860352319495#heading-12)

* [Flatmap](https://juejin.im/post/6844903860352319495#heading-13)

	* [Flatmap on array](https://juejin.im/post/6844903860352319495#heading-14)
	* [Flatmap on array of dictionaries](https://juejin.im/post/6844903860352319495#heading-15)
	* [Flatmap on set](https://juejin.im/post/6844903860352319495#heading-16)
	* [Flatmap by filtering or mapping](https://juejin.im/post/6844903860352319495#heading-17)

* [链式 : (map + filter + reduce)](https://juejin.im/post/6844903860352319495#heading-18)
* [CompactMap](https://juejin.im/post/6844903860352319495#heading-19)

	* [Tip](https://juejin.im/post/6844903860352319495#heading-20)

* [总结](https://juejin.im/post/6844903860352319495#heading-21)

分享

| 掘金浏览器插件 - 打开新标签，发现好内容
| 网罗十几家优质内容源，无论你是程序员，产品经理或是设计师，每天都可以在这里发现优质内容。

[juejin.im](https://juejin.im/post/6844903860352319495)