---
title: Swift编程语言中的optional(可选)
author: admin
type: post
date: 2014-06-11T04:08:44+00:00
url: /archives/15046
categories:
 - 程序开发
tags:
 - swift

---
## 可选 {#-}

使用可选来处理值可能缺失的情况。可选表示：

- 有值，等于 x


或者

- 没有值


> 注意：C 和 Objective-C 中并没有可选这个概念。最接近的是 Objective-C 中的一个特性，一个方法要不返回一个对象要不返回 `nil`， `nil` 表示“缺少一个合法的对象”。然而，这只对对象起作用——对于结构体，基本的 C 类型或者枚举类型不起作用。对于这些类型，Objective-C 方法一般会返回一个特殊值（比如 `NSNotFound`）来暗示值缺失。这种方法假设方法的调用者知道并记得对特殊值进行判断。然而，Swift 的可选可以让你暗示任意类型的值缺失，并不需要一个特殊值。

来看一个例子。Swift 的 `String` 类型有一个叫做 `toInt` 的方法，作用是将一个 `String` 值转换成一个 `Int` 值。然而，并不是所有的字符串都可以转换成一个整数。字符串 `"123"` 可以被转换成数字 `123`，但是字符串 `"hello, world"` 不行。

下面的例子使用 `toInt` 方法来尝试将一个 `String` 转换成 `Int`：

 let possibleNumber = "123"
 let convertedNumber = possibleNumber.toInt()
 // convertedNumber 被推测为类型 "Int?"， 或者类型 "optional Int"


因为 `toInt` 方法可能会失败，所以它返回一个可选的 `Int`，而不是一个 `Int`。一个可选的 `Int` 被写作 `Int?` 而不是 `Int`。问号暗示包含的值是可选，也就是说可能包含 `Int` 值也可能不包含值。（不能包含其他任何值比如 `Bool` 值或者 `String` 值。只能是 `Int` 或者什么都没有。）

### if 语句以及强制解析

你可以使用 `if` 语句来判断一个可选是否包含值。如果可选有值，结果是 `true`；如果没有值，结果是 `false`。

当你确定可选包含值之后，你可以在可选的名字后面加一个 `!` 来获取值。这个惊叹号表示“我知道这个可选有值，请使用它。”这被称为可选值的强制解析：

 if convertedNumber {
 println("\(possibleNumber) has an integer value of \(convertedNumber!)")
 } else {
 println("\(possibleNumber) could not be converted to an integer")
 }
 // 输出 "123 has an integer value of 123"


更多关于 `if` 语句的内容参见 `控制流(待添加链接)`。

> 注意：使用 `!` 来获取一个不存在的可选值会导致运行时错误。。使用 `!` 来强制解析值之前，一定要确定可选包含一个非 `nil` 的值。

### 可选绑定

使用可选绑定来判断可选是否包含值，如果包含就把值赋给一个临时常量或者变量。可选绑定可以用在 `if` 和 `while` 语句中来对可选的值进行判断并把值赋给一个常量或者变量。 `if` 和 `while` 语句详情参见 `控制流`。

像下面这样写一个可选绑定：

 if let constantName = someOptional {
 statements
 }


你可以像上面这样使用可选绑定来重写 `possibleNumber` 这个例子：

 if let actualNumber = possibleNumber.toInt() {
 println("\(possibleNumber) has an integer value of \(actualNumber)")
 } else {
 println("\(possibleNumber) could not be converted to an integer")
 }
 // 输出 "123 has an integer value of 123"


这段代码可以被理解为：

“如果 `possibleNumber.toInt` 返回的可选 `Int` 包含一个值，创建一个叫做 `actualNumber` 的新常量并将可选包含的值赋给它。”

如果转换成功， `actualNumber` 常量可以在 `if` 语句的第一个分支中使用。它已经被可选包含的值初始化过，所以不需要再使用 `!` 后缀来获取它的值。在这个例子中， `actualNumber` 只被用来输出转换结果。

你可以在可选绑定中使用常量和变量。如果你想在 `if` 语句的第一个分支中操作 `actualNumber` 的值，你可以改成 `if var actualNumber`，这样可选包含的值就会被赋给一个变量。

### nil

你可以给可选变量赋值为 `nil` 来表示它没有值：

 var serverResponseCode: Int? = 404
 // serverResponseCode 包含一个可选的 Int 值 404
 serverResponseCode = nil
 // serverResponseCode 现在不包含值


> 注意： `nil` 不能用于非可选的常量和变量。如果你的代码中有常量或者变量需要处理值缺失的情况，请把它们声明成对应的可选类型。

如果你声明一个可选常量或者变量但是没有赋值，它们会自动被设置为 `nil`：

 var surveyAnswer: String?
 // surveyAnswer 被自动设置为 nil


> 注意：Swift 的 `nil` 和 Objective-C 中的 `nil` 并不一样。在 Objective-C 中， `nil` 是一个指向不存在对象的指针。在 Swift 中， `nil` 不是指针——它是一个确定的值，用来表示值缺失。任何类型的可选都可以被设置为 `nil`，不只是对象类型。

### 隐式解析可选

如上所述，可选暗示了常量或者变量可以“没有值”。可选可以通过 `if` 语句来判断是否有值，如果有值的话可以通过可选绑定来解析值。

有时候在程序架构中，第一次被赋值之后，可以确定一个可选总会有值。在这种情况下，每次都要判断和解析可选值是非常低效的，因为可以确定它总会有值。

这种类型的可选被定义为隐式解析可选。把后缀 `?` 改成 `!` 来声明一个隐式解析可选，比如 `String!`。

当可选被第一次赋值之后就可以确定之后一直有值的时候，隐式解析可选非常有用。隐式解析可选主要被用在 Swift 中类的构造过程中，详情参见 `无主引用和隐式解析可选属性(Unowned References and Implicitly Unwrapped Optional Properties待添加链接)`。

一个隐式解析可选其实就是一个普通的可选，但是可以被当做非可选来使用，并不需要每次都使用解析来获取可选值。下面的例子展示了可选 `String` 和隐式解析可选 `String` 之间的区别：

 let possibleString: String? = "An optional string."
 println(possibleString!) // 需要惊叹号来获取值
 // 输出 "An optional string."

 let assumedString: String! = "An implicitly unwrapped optional string."
 println(assumedString) // 不需要惊叹号
 // 输出 "An implicitly unwrapped optional string."


你可以把隐式解析可选当做一个可以自动解析的可选。你要做的只是声明的时候把惊叹号放到类型的结尾，而不是每次获取值的变量结尾。

> 注意：如果你在隐式解析可选没有值的时候尝试获取，会触发运行时错误。和你在没有值的普通可选后面加一个惊叹号一样。

你仍然可以把隐式解析可选当做普通可选来判断它是否包含值： if assumedString { println(assumedString) } // 输出 “An implicitly unwrapped optional string.”

你也可以在可选绑定中使用隐式解析可选来检查并解析它的值： if let definiteString = assumedString { println(definiteString) } // 输出 “An implicitly unwrapped optional string.”

> 注意：如果一个变量之后可能变成 `nil` 的话请不要使用隐式解析可选。如果你需要在变量的生命周期中判断是否是 `nil` 的话，请使用普通可选类型。

转自： [http://numbbbbb.github.io/the-swift-programming-language-in-chinese/chapter2/01_The_Basics.html](http://numbbbbb.github.io/the-swift-programming-language-in-chinese/chapter2/01_The_Basics.html)