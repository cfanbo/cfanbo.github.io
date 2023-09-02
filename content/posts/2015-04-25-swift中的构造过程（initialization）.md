---
title: swift中的构造过程（Initialization）
author: admin
type: post
date: 2015-04-24T19:45:24+00:00
url: /archives/15660
categories:
 - 程序开发
tags:
 - swift

---
[http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/14_Initialization.html](http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/14_Initialization.html)

本页包含内容：

 * [存储型属性的初始赋值][1]
 * [定制化构造过程][2]
 * [默认构造器][3]
 * [值类型的构造器代理][4]
 * [类的继承和构造过程][5]
 * [可失败构造器][6]
 * [必要构造器][7]
 * [通过闭包和函数来设置属性的默认值][8]

构造过程是为了使用某个类、结构体或枚举类型的实例而进行的准备过程。这个过程包含了为实例中的每个属性设置初始值和为其执行必要的准备和初始化任务。

构造过程是通过定义构造器（`Initializers`）来实现的，这些构造器可以看做是用来创建特定类型实例的特殊方法。与 Objective-C 中的构造器不同，Swift 的构造器无需返回值，它们的主要任务是保证新实例在第一次使用前完成正确的初始化。

类实例也可以通过定义析构器（`deinitializer`）在类实例释放之前执行特定的清除工作。想了解更多关于析构器的内容，请参考[析构过程][9]。

## 存储型属性的初始赋值 {#%E5%AD%98%E5%82%A8%E5%9E%8B%E5%B1%9E%E6%80%A7%E7%9A%84%E5%88%9D%E5%A7%8B%E8%B5%8B%E5%80%BC}

类和结构体在实例创建时，必须为所有存储型属性设置合适的初始值。存储型属性的值不能处于一个未知的状态。

你可以在构造器中为存储型属性赋初值，也可以在定义属性时为其设置默认值。以下章节将详细介绍这两种方法。

> 注意：
> 当你为存储型属性设置默认值或者在构造器中为其赋值时，它们的值是被直接设置的，不会触发任何属性观测器（`property observers`）。

### 构造器 {#%E6%9E%84%E9%80%A0%E5%99%A8}

构造器在创建某特定类型的新实例时调用。它的最简形式类似于一个不带任何参数的实例方法，以关键字`init`命名。

下面例子中定义了一个用来保存华氏温度的结构体`Fahrenheit`，它拥有一个`Double`类型的存储型属性`temperature`：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">Fahrenheit</span> </span>{
    <span class="hljs-keyword">var</span> temperature: <span class="hljs-type">Double</span>
    <span class="hljs-keyword">init</span>() {
        temperature = <span class="hljs-number">32.0</span>
    }
}

```

```lang-swift
<span class="hljs-keyword">var</span> f = <span class="hljs-type">Fahrenheit</span>()
<span class="hljs-built_in">println</span>(<span class="hljs-string">"The default temperature is <span class="hljs-subst">\(f.temperature)</span>° Fahrenheit"</span>)
<span class="hljs-comment">// 输出 "The default temperature is 32.0° Fahrenheit”</span>

```

这个结构体定义了一个不带参数的构造器`init`，并在里面将存储型属性`temperature`的值初始化为`32.0`（华摄氏度下水的冰点）。

### 默认属性值 {#%E9%BB%98%E8%AE%A4%E5%B1%9E%E6%80%A7%E5%80%BC}

如前所述，你可以在构造器中为存储型属性设置初始值；同样，你也可以在属性声明时为其设置默认值。

> 注意：
> 如果一个属性总是使用同一个初始值，可以为其设置一个默认值。无论定义默认值还是在构造器中赋值，最终它们实现的效果是一样的，只不过默认值将属性的初始化和属性的声明结合的更紧密。使用默认值能让你的构造器更简洁、更清晰，且能通过默认值自动推导出属性的类型；同时，它也能让你充分利用默认构造器、构造器继承（后续章节将讲到）等特性。

你可以使用更简单的方式在定义结构体`Fahrenheit`时为属性`temperature`设置默认值：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">Fahrenheit</span> </span>{
    <span class="hljs-keyword">var</span> temperature = <span class="hljs-number">32.0</span>
}

```

## 定制化构造过程 {#%E5%AE%9A%E5%88%B6%E5%8C%96%E6%9E%84%E9%80%A0%E8%BF%87%E7%A8%8B}

你可以通过输入参数和可选属性类型来定制构造过程，也可以在构造过程中修改常量属性。这些都将在后面章节中提到。

### 构造参数 {#%E6%9E%84%E9%80%A0%E5%8F%82%E6%95%B0}

你可以在定义构造器时提供构造参数，为其提供定制化构造所需值的类型和名字。构造器参数的功能和语法跟函数和方法参数相同。

下面例子中定义了一个包含摄氏度温度的结构体`Celsius`。它定义了两个不同的构造器：`init(fromFahrenheit:)`和`init(fromKelvin:)`，二者分别通过接受不同刻度表示的温度值来创建新的实例：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">Celsius</span> </span>{
    <span class="hljs-keyword">var</span> temperatureInCelsius: <span class="hljs-type">Double</span> = <span class="hljs-number">0.0</span>
    <span class="hljs-keyword">init</span>(fromFahrenheit fahrenheit: <span class="hljs-type">Double</span>) {
        temperatureInCelsius = (fahrenheit - <span class="hljs-number">32.0</span>) / <span class="hljs-number">1.8</span>
    }
    <span class="hljs-keyword">init</span>(fromKelvin kelvin: <span class="hljs-type">Double</span>) {
        temperatureInCelsius = kelvin - <span class="hljs-number">273.15</span>
    }
}

```

```lang-swift
<span class="hljs-keyword">let</span> boilingPointOfWater = <span class="hljs-type">Celsius</span>(fromFahrenheit: <span class="hljs-number">212.0</span>)
<span class="hljs-comment">// boilingPointOfWater.temperatureInCelsius 是 100.0</span>
<span class="hljs-keyword">let</span> freezingPointOfWater = <span class="hljs-type">Celsius</span>(fromKelvin: <span class="hljs-number">273.15</span>)
<span class="hljs-comment">// freezingPointOfWater.temperatureInCelsius 是 0.0”</span>

```

第一个构造器拥有一个构造参数，其外部名字为`fromFahrenheit`，内部名字为`fahrenheit`；第二个构造器也拥有一个构造参数，其外部名字为`fromKelvin`，内部名字为`kelvin`。这两个构造器都将唯一的参数值转换成摄氏温度值，并保存在属性`temperatureInCelsius`中。

### 内部和外部参数名 {#%E5%86%85%E9%83%A8%E5%92%8C%E5%A4%96%E9%83%A8%E5%8F%82%E6%95%B0%E5%90%8D}

跟函数和方法参数相同，构造参数也存在一个在构造器内部使用的参数名字和一个在调用构造器时使用的外部参数名字。

然而，构造器并不像函数和方法那样在括号前有一个可辨别的名字。所以在调用构造器时，主要通过构造器中的参数名和类型来确定需要调用的构造器。正因为参数如此重要，如果你在定义构造器时没有提供参数的外部名字，Swift 会为每个构造器的参数自动生成一个跟内部名字相同的外部名，就相当于在每个构造参数之前加了一个哈希符号。

> 注意：
> 如果你不希望为构造器的某个参数提供外部名字，你可以使用下划线`_`来显示描述它的外部名，以此覆盖上面所说的默认行为。

以下例子中定义了一个结构体`Color`，它包含了三个常量：`red`、`green`和`blue`。这些属性可以存储0.0到1.0之间的值，用来指示颜色中红、绿、蓝成分的含量。

`Color`提供了一个构造器，其中包含三个`Double`类型的构造参数：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">Color</span> </span>{
    <span class="hljs-keyword">let</span> red = <span class="hljs-number">0.0</span>, green = <span class="hljs-number">0.0</span>, blue = <span class="hljs-number">0.0</span>
    <span class="hljs-keyword">init</span>(red: <span class="hljs-type">Double</span>, green: <span class="hljs-type">Double</span>, blue: <span class="hljs-type">Double</span>) {
        <span class="hljs-keyword">self</span>.red   = red
        <span class="hljs-keyword">self</span>.green = green
        <span class="hljs-keyword">self</span>.blue  = blue
    }
}

```

每当你创建一个新的`Color`实例，你都需要通过三种颜色的外部参数名来传值，并调用构造器。

```lang-swift
<span class="hljs-keyword">let</span> magenta = <span class="hljs-type">Color</span>(red: <span class="hljs-number">1.0</span>, green: <span class="hljs-number">0.0</span>, blue: <span class="hljs-number">1.0</span>)

```

注意，如果不通过外部参数名字传值，你是没法调用这个构造器的。只要构造器定义了某个外部参数名，你就必须使用它，忽略它将导致编译错误：

```lang-swift
<span class="hljs-keyword">let</span> veryGreen = <span class="hljs-type">Color</span>(<span class="hljs-number">0.0</span>, <span class="hljs-number">1.0</span>, <span class="hljs-number">0.0</span>)
<span class="hljs-comment">// 报编译时错误，需要外部名称</span>

```

### 可选属性类型 {#%E5%8F%AF%E9%80%89%E5%B1%9E%E6%80%A7%E7%B1%BB%E5%9E%8B}

如果你定制的类型包含一个逻辑上允许取值为空的存储型属性–不管是因为它无法在初始化时赋值，还是因为它可以在之后某个时间点可以赋值为空–你都需要将它定义为可选类型`optional type`。可选类型的属性将自动初始化为空`nil`，表示这个属性是故意在初始化时设置为空的。

下面例子中定义了类`SurveyQuestion`，它包含一个可选字符串属性`response`：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SurveyQuestion</span> </span>{
    <span class="hljs-keyword">var</span> text: <span class="hljs-type">String</span>
    <span class="hljs-keyword">var</span> response: <span class="hljs-type">String</span>?
    <span class="hljs-keyword">init</span>(text: <span class="hljs-type">String</span>) {
        <span class="hljs-keyword">self</span>.text = text
    }
    <span class="hljs-func"><span class="hljs-keyword">func</span> <span class="hljs-title">ask</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-built_in">println</span>(text)
    }
}
<span class="hljs-keyword">let</span> cheeseQuestion = <span class="hljs-type">SurveyQuestion</span>(text: <span class="hljs-string">"Do you like cheese?"</span>)
cheeseQuestion.ask()
<span class="hljs-comment">// 输出 "Do you like cheese?"</span>
cheeseQuestion.response = <span class="hljs-string">"Yes, I do like cheese."</span>

```

调查问题在问题提出之后，我们才能得到回答。所以我们将属性回答`response`声明为`String?`类型，或者说是可选字符串类型`optional String`。当`SurveyQuestion`实例化时，它将自动赋值为空`nil`，表明暂时还不存在此字符串。

### 构造过程中常量属性的修改 {#%E6%9E%84%E9%80%A0%E8%BF%87%E7%A8%8B%E4%B8%AD%E5%B8%B8%E9%87%8F%E5%B1%9E%E6%80%A7%E7%9A%84%E4%BF%AE%E6%94%B9}

只要在构造过程结束前常量的值能确定，你可以在构造过程中的任意时间点修改常量属性的值。

> 注意：
> 对某个类实例来说，它的常量属性只能在定义它的类的构造过程中修改；不能在子类中修改。

你可以修改上面的`SurveyQuestion`示例，用常量属性替代变量属性`text`，指明问题内容`text`在其创建之后不会再被修改。尽管`text`属性现在是常量，我们仍然可以在其类的构造器中设置它的值：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SurveyQuestion</span> </span>{
    <span class="hljs-keyword">let</span> text: <span class="hljs-type">String</span>
    <span class="hljs-keyword">var</span> response: <span class="hljs-type">String</span>?
    <span class="hljs-keyword">init</span>(text: <span class="hljs-type">String</span>) {
        <span class="hljs-keyword">self</span>.text = text
    }
    <span class="hljs-func"><span class="hljs-keyword">func</span> <span class="hljs-title">ask</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-built_in">println</span>(text)
    }
}
<span class="hljs-keyword">let</span> beetsQuestion = <span class="hljs-type">SurveyQuestion</span>(text: <span class="hljs-string">"How about beets?"</span>)
beetsQuestion.ask()
<span class="hljs-comment">// 输出 "How about beets?"</span>
beetsQuestion.response = <span class="hljs-string">"I also like beets. (But not with cheese.)"</span>

```

## 默认构造器 {#%E9%BB%98%E8%AE%A4%E6%9E%84%E9%80%A0%E5%99%A8}

Swift 将为所有属性已提供默认值的且自身没有定义任何构造器的结构体或基类，提供一个默认的构造器。这个默认构造器将简单的创建一个所有属性值都设置为默认值的实例。

下面例子中创建了一个类`ShoppingListItem`，它封装了购物清单中的某一项的属性：名字（`name`）、数量（`quantity`）和购买状态 `purchase state`。

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ShoppingListItem</span> </span>{
    <span class="hljs-keyword">var</span> name: <span class="hljs-type">String</span>?
    <span class="hljs-keyword">var</span> quantity = <span class="hljs-number">1</span>
    <span class="hljs-keyword">var</span> purchased = <span class="hljs-built_in">false</span>
}
<span class="hljs-keyword">var</span> item = <span class="hljs-type">ShoppingListItem</span>()

```

由于`ShoppingListItem`类中的所有属性都有默认值，且它是没有父类的基类，它将自动获得一个可以为所有属性设置默认值的默认构造器（尽管代码中没有显式为`name`属性设置默认值，但由于`name`是可选字符串类型，它将默认设置为`nil`）。上面例子中使用默认构造器创造了一个`ShoppingListItem`类的实例（使用`ShoppingListItem()`形式的构造器语法），并将其赋值给变量`item`。

### 结构体的逐一成员构造器 {#%E7%BB%93%E6%9E%84%E4%BD%93%E7%9A%84%E9%80%90%E4%B8%80%E6%88%90%E5%91%98%E6%9E%84%E9%80%A0%E5%99%A8}

除上面提到的默认构造器，如果结构体对所有存储型属性提供了默认值且自身没有提供定制的构造器，它们能自动获得一个逐一成员构造器。

逐一成员构造器是用来初始化结构体新实例里成员属性的快捷方法。我们在调用逐一成员构造器时，通过与成员属性名相同的参数名进行传值来完成对成员属性的初始赋值。

下面例子中定义了一个结构体`Size`，它包含两个属性`width`和`height`。Swift 可以根据这两个属性的初始赋值`0.0`自动推导出它们的类型`Double`。

由于这两个存储型属性都有默认值，结构体`Size`自动获得了一个逐一成员构造器 `init(width:height:)`。 你可以用它来为`Size`创建新的实例：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">Size</span> </span>{
    <span class="hljs-keyword">var</span> width = <span class="hljs-number">0.0</span>, height = <span class="hljs-number">0.0</span>
}
<span class="hljs-keyword">let</span> twoByTwo = <span class="hljs-type">Size</span>(width: <span class="hljs-number">2.0</span>, height: <span class="hljs-number">2.0</span>)

```

## 值类型的构造器代理 {#%E5%80%BC%E7%B1%BB%E5%9E%8B%E7%9A%84%E6%9E%84%E9%80%A0%E5%99%A8%E4%BB%A3%E7%90%86}

构造器可以通过调用其它构造器来完成实例的部分构造过程。这一过程称为构造器代理，它能减少多个构造器间的代码重复。

构造器代理的实现规则和形式在值类型和类类型中有所不同。值类型（结构体和枚举类型）不支持继承，所以构造器代理的过程相对简单，因为它们只能代理给本身提供的其它构造器。类则不同，它可以继承自其它类（请参考[继承][10]），这意味着类有责任保证其所有继承的存储型属性在构造时也能正确的初始化。这些责任将在后续章节[类的继承和构造过程][5]中介绍。

对于值类型，你可以使用`self.init`在自定义的构造器中引用其它的属于相同值类型的构造器。并且你只能在构造器内部调用`self.init`。

注意，如果你为某个值类型定义了一个定制的构造器，你将无法访问到默认构造器（如果是结构体，则无法访问逐一对象构造器）。这个限制可以防止你在为值类型定义了一个更复杂的，完成了重要准备构造器之后，别人还是错误的使用了那个自动生成的构造器。

> 注意：
> 假如你想通过默认构造器、逐一对象构造器以及你自己定制的构造器为值类型创建实例，我们建议你将自己定制的构造器写到扩展（`extension`）中，而不是跟值类型定义混在一起。想查看更多内容，请查看[扩展][11]章节。

下面例子将定义一个结构体`Rect`，用来代表几何矩形。这个例子需要两个辅助的结构体`Size`和`Point`，它们各自为其所有的属性提供了初始值`0.0`。

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">Size</span> </span>{
    <span class="hljs-keyword">var</span> width = <span class="hljs-number">0.0</span>, height = <span class="hljs-number">0.0</span>
}
<span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">Point</span> </span>{
    <span class="hljs-keyword">var</span> x = <span class="hljs-number">0.0</span>, y = <span class="hljs-number">0.0</span>
}

```

你可以通过以下三种方式为`Rect`创建实例–使用默认的0值来初始化`origin`和`size`属性；使用特定的`origin`和`size`实例来初始化；使用特定的`center`和`size`来初始化。在下面`Rect`结构体定义中，我们为这三种方式提供了三个自定义的构造器：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">Rect</span> </span>{
    <span class="hljs-keyword">var</span> origin = <span class="hljs-type">Point</span>()
    <span class="hljs-keyword">var</span> size = <span class="hljs-type">Size</span>()
    <span class="hljs-keyword">init</span>() {}
    <span class="hljs-keyword">init</span>(origin: <span class="hljs-type">Point</span>, size: <span class="hljs-type">Size</span>) {
        <span class="hljs-keyword">self</span>.origin = origin
        <span class="hljs-keyword">self</span>.size = size
    }
    <span class="hljs-keyword">init</span>(center: <span class="hljs-type">Point</span>, size: <span class="hljs-type">Size</span>) {
        <span class="hljs-keyword">let</span> originX = center.x - (size.width / <span class="hljs-number">2</span>)
        <span class="hljs-keyword">let</span> originY = center.y - (size.height / <span class="hljs-number">2</span>)
        <span class="hljs-keyword">self</span>.<span class="hljs-keyword">init</span>(origin: <span class="hljs-type">Point</span>(x: originX, y: originY), size: size)
    }
}

```

第一个`Rect`构造器`init()`，在功能上跟没有自定义构造器时自动获得的默认构造器是一样的。这个构造器是一个空函数，使用一对大括号`{}`来描述，它没有执行任何定制的构造过程。调用这个构造器将返回一个`Rect`实例，它的`origin`和`size`属性都使用定义时的默认值`Point(x: 0.0, y: 0.0)`和`Size(width: 0.0, height: 0.0)`：

```lang-swift
<span class="hljs-keyword">let</span> basicRect = <span class="hljs-type">Rect</span>()
<span class="hljs-comment">// basicRect 的原点是 (0.0, 0.0)，尺寸是 (0.0, 0.0)</span>

```

第二个`Rect`构造器`init(origin:size:)`，在功能上跟结构体在没有自定义构造器时获得的逐一成员构造器是一样的。这个构造器只是简单地将`origin`和`size`的参数值赋给对应的存储型属性：

```lang-swift
<span class="hljs-keyword">let</span> originRect = <span class="hljs-type">Rect</span>(origin: <span class="hljs-type">Point</span>(x: <span class="hljs-number">2.0</span>, y: <span class="hljs-number">2.0</span>),
    size: <span class="hljs-type">Size</span>(width: <span class="hljs-number">5.0</span>, height: <span class="hljs-number">5.0</span>))
<span class="hljs-comment">// originRect 的原点是 (2.0, 2.0)，尺寸是 (5.0, 5.0)</span>

```

第三个`Rect`构造器`init(center:size:)`稍微复杂一点。它先通过`center`和`size`的值计算出`origin`的坐标。然后再调用（或代理给）`init(origin:size:)`构造器来将新的`origin`和`size`值赋值到对应的属性中：

```lang-swift
<span class="hljs-keyword">let</span> centerRect = <span class="hljs-type">Rect</span>(center: <span class="hljs-type">Point</span>(x: <span class="hljs-number">4.0</span>, y: <span class="hljs-number">4.0</span>),
    size: <span class="hljs-type">Size</span>(width: <span class="hljs-number">3.0</span>, height: <span class="hljs-number">3.0</span>))
<span class="hljs-comment">// centerRect 的原点是 (2.5, 2.5)，尺寸是 (3.0, 3.0)</span>

```

构造器`init(center:size:)`可以自己将`origin`和`size`的新值赋值到对应的属性中。然而尽量利用现有的构造器和它所提供的功能来实现`init(center:size:)`的功能，是更方便、更清晰和更直观的方法。

> 注意：
> 如果你想用另外一种不需要自己定义`init()`和`init(origin:size:)`的方式来实现这个例子，请参考[扩展][11]。

## 类的继承和构造过程 {#%E7%B1%BB%E7%9A%84%E7%BB%A7%E6%89%BF%E5%92%8C%E6%9E%84%E9%80%A0%E8%BF%87%E7%A8%8B}

类里面的所有存储型属性–包括所有继承自父类的属性–都必须在构造过程中设置初始值。

Swift 提供了两种类型的类构造器来确保所有类实例中存储型属性都能获得初始值，它们分别是指定构造器和便利构造器。

### 指定构造器和便利构造器 {#%E6%8C%87%E5%AE%9A%E6%9E%84%E9%80%A0%E5%99%A8%E5%92%8C%E4%BE%BF%E5%88%A9%E6%9E%84%E9%80%A0%E5%99%A8}

指定构造器是类中最主要的构造器。一个指定构造器将初始化类中提供的所有属性，并根据父类链往上调用父类的构造器来实现父类的初始化。

每一个类都必须拥有至少一个指定构造器。在某些情况下，许多类通过继承了父类中的指定构造器而满足了这个条件。具体内容请参考后续章节[自动构造器的继承][12]。

便利构造器是类中比较次要的、辅助型的构造器。你可以定义便利构造器来调用同一个类中的指定构造器，并为其参数提供默认值。你也可以定义便利构造器来创建一个特殊用途或特定输入的实例。

你应当只在必要的时候为类提供便利构造器，比方说某种情况下通过使用便利构造器来快捷调用某个指定构造器，能够节省更多开发时间并让类的构造过程更清晰明了。

### 构造器链 {#%E6%9E%84%E9%80%A0%E5%99%A8%E9%93%BE}

为了简化指定构造器和便利构造器之间的调用关系，Swift 采用以下三条规则来限制构造器之间的代理调用：

#### 规则 1 {#%E8%A7%84%E5%88%99-1}

指定构造器必须调用其直接父类的的指定构造器。

#### 规则 2 {#%E8%A7%84%E5%88%99-2}

便利构造器必须调用同一类中定义的其它构造器。

#### 规则 3 {#%E8%A7%84%E5%88%99-3}

便利构造器必须最终以调用一个指定构造器结束。

一个更方便记忆的方法是：

 * 指定构造器必须总是向上代理
 * 便利构造器必须总是横向代理

这些规则可以通过下面图例来说明：

![构造器代理图](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/Art/initializerDelegation01_2x.png)

如图所示，父类中包含一个指定构造器和两个便利构造器。其中一个便利构造器调用了另外一个便利构造器，而后者又调用了唯一的指定构造器。这满足了上面提到的规则2和3。这个父类没有自己的父类，所以规则1没有用到。

子类中包含两个指定构造器和一个便利构造器。便利构造器必须调用两个指定构造器中的任意一个，因为它只能调用同一个类里的其他构造器。这满足了上面提到的规则2和3。而两个指定构造器必须调用父类中唯一的指定构造器，这满足了规则1。

> 注意：
> 这些规则不会影响使用时，如何用类去创建实例。任何上图中展示的构造器都可以用来完整创建对应类的实例。这些规则只在实现类的定义时有影响。

下面图例中展示了一种针对四个类的更复杂的类层级结构。它演示了指定构造器是如何在类层级中充当“管道”的作用，在类的构造器链上简化了类之间的相互关系。

![复杂构造器代理图](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/Art/initializerDelegation02_2x.png)

### 两段式构造过程 {#%E4%B8%A4%E6%AE%B5%E5%BC%8F%E6%9E%84%E9%80%A0%E8%BF%87%E7%A8%8B}

Swift 中类的构造过程包含两个阶段。第一个阶段，每个存储型属性通过引入它们的类的构造器来设置初始值。当每一个存储型属性值被确定后，第二阶段开始，它给每个类一次机会在新实例准备使用之前进一步定制它们的存储型属性。

两段式构造过程的使用让构造过程更安全，同时在整个类层级结构中给予了每个类完全的灵活性。两段式构造过程可以防止属性值在初始化之前被访问；也可以防止属性被另外一个构造器意外地赋予不同的值。

> 注意：
> Swift的两段式构造过程跟 Objective-C 中的构造过程类似。最主要的区别在于阶段 1，Objective-C 给每一个属性赋值``或空值（比如说``或`nil`）。Swift 的构造流程则更加灵活，它允许你设置定制的初始值，并自如应对某些属性不能以``或`nil`作为合法默认值的情况。

Swift 编译器将执行 4 种有效的安全检查，以确保两段式构造过程能顺利完成：

#### 安全检查 1 {#%E5%AE%89%E5%85%A8%E6%A3%80%E6%9F%A5-1}

指定构造器必须保证它所在类引入的所有属性都必须先初始化完成，之后才能将其它构造任务向上代理给父类中的构造器。

如上所述，一个对象的内存只有在其所有存储型属性确定之后才能完全初始化。为了满足这一规则，指定构造器必须保证它所在类引入的属性在它往上代理之前先完成初始化。

#### 安全检查 2 {#%E5%AE%89%E5%85%A8%E6%A3%80%E6%9F%A5-2}

指定构造器必须先向上代理调用父类构造器，然后再为继承的属性设置新值。如果没这么做，指定构造器赋予的新值将被父类中的构造器所覆盖。

#### 安全检查 3 {#%E5%AE%89%E5%85%A8%E6%A3%80%E6%9F%A5-3}

便利构造器必须先代理调用同一类中的其它构造器，然后再为任意属性赋新值。如果没这么做，便利构造器赋予的新值将被同一类中其它指定构造器所覆盖。

#### 安全检查 4 {#%E5%AE%89%E5%85%A8%E6%A3%80%E6%9F%A5-4}

构造器在第一阶段构造完成之前，不能调用任何实例方法、不能读取任何实例属性的值，`self`的值不能被引用。

类实例在第一阶段结束以前并不是完全有效，仅能访问属性和调用方法，一旦完成第一阶段，该实例才会声明为有效实例。

以下是两段式构造过程中基于上述安全检查的构造流程展示：

#### 阶段 1 {#%E9%98%B6%E6%AE%B5-1}

 * 某个指定构造器或便利构造器被调用；
 * 完成新实例内存的分配，但此时内存还没有被初始化；
 * 指定构造器确保其所在类引入的所有存储型属性都已赋初值。存储型属性所属的内存完成初始化；
 * 指定构造器将调用父类的构造器，完成父类属性的初始化；
 * 这个调用父类构造器的过程沿着构造器链一直往上执行，直到到达构造器链的最顶部；
 * 当到达了构造器链最顶部，且已确保所有实例包含的存储型属性都已经赋值，这个实例的内存被认为已经完全初始化。此时阶段1完成。

#### 阶段 2 {#%E9%98%B6%E6%AE%B5-2}

 * 从顶部构造器链一直往下，每个构造器链中类的指定构造器都有机会进一步定制实例。构造器此时可以访问`self`、修改它的属性并调用实例方法等等。
 * 最终，任意构造器链中的便利构造器可以有机会定制实例和使用`self`。

下图展示了在假定的子类和父类之间构造的阶段1： ·![构造过程阶段1](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/Art/twoPhaseInitialization01_2x.png)

在这个例子中，构造过程从对子类中一个便利构造器的调用开始。这个便利构造器此时没法修改任何属性，它把构造任务代理给同一类中的指定构造器。

如安全检查1所示，指定构造器将确保所有子类的属性都有值。然后它将调用父类的指定构造器，并沿着造器链一直往上完成父类的构建过程。

父类中的指定构造器确保所有父类的属性都有值。由于没有更多的父类需要构建，也就无需继续向上做构建代理。

一旦父类中所有属性都有了初始值，实例的内存被认为是完全初始化，而阶段1也已完成。

以下展示了相同构造过程的阶段2：

![构建过程阶段2](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/Art/twoPhaseInitialization02_2x.png)

父类中的指定构造器现在有机会进一步来定制实例（尽管它没有这种必要）。

一旦父类中的指定构造器完成调用，子类的构指定构造器可以执行更多的定制操作（同样，它也没有这种必要）。

最终，一旦子类的指定构造器完成调用，最开始被调用的便利构造器可以执行更多的定制操作。

### 构造器的继承和重载 {#%E6%9E%84%E9%80%A0%E5%99%A8%E7%9A%84%E7%BB%A7%E6%89%BF%E5%92%8C%E9%87%8D%E8%BD%BD}

跟 Objective-C 中的子类不同，Swift 中的子类不会默认继承父类的构造器。Swift 的这种机制可以防止一个父类的简单构造器被一个更专业的子类继承，并被错误的用来创建子类的实例。

假如你希望自定义的子类中能实现一个或多个跟父类相同的构造器–也许是为了完成一些定制的构造过程–你可以在你定制的子类中提供和重载与父类相同的构造器。

如果你重载的构造器是一个指定构造器，你可以在子类里重载它的实现，并在自定义版本的构造器中调用父类版本的构造器。

如果你重载的构造器是一个便利构造器，你的重载过程必须通过调用同一类中提供的其它指定构造器来实现。这一规则的详细内容请参考[构造器链][13]。

> 注意：
> 与方法、属性和下标不同，在重载构造器时你没有必要使用关键字`override`。

### 自动构造器的继承 {#%E8%87%AA%E5%8A%A8%E6%9E%84%E9%80%A0%E5%99%A8%E7%9A%84%E7%BB%A7%E6%89%BF}

如上所述，子类不会默认继承父类的构造器。但是如果特定条件可以满足，父类构造器是可以被自动继承的。在实践中，这意味着对于许多常见场景你不必重载父类的构造器，并且在尽可能安全的情况下以最小的代价来继承父类的构造器。

假设要为子类中引入的任意新属性提供默认值，请遵守以下2个规则：

#### 规则 1 {#%E8%A7%84%E5%88%99-1}

如果子类没有定义任何指定构造器，它将自动继承所有父类的指定构造器。

#### 规则 2 {#%E8%A7%84%E5%88%99-2}

如果子类提供了所有父类指定构造器的实现–不管是通过规则1继承过来的，还是通过自定义实现的–它将自动继承所有父类的便利构造器。

即使你在子类中添加了更多的便利构造器，这两条规则仍然适用。

> 注意：
> 子类可以通过部分满足规则2的方式，使用子类便利构造器来实现父类的指定构造器。

### 指定构造器和便利构造器的语法 {#%E6%8C%87%E5%AE%9A%E6%9E%84%E9%80%A0%E5%99%A8%E5%92%8C%E4%BE%BF%E5%88%A9%E6%9E%84%E9%80%A0%E5%99%A8%E7%9A%84%E8%AF%AD%E6%B3%95}

类的指定构造器的写法跟值类型简单构造器一样：

```lang-swift
<span class="hljs-keyword">init</span>(parameters) {
    statements
}

```

便利构造器也采用相同样式的写法，但需要在`init`关键字之前放置`convenience`关键字，并使用空格将它们俩分开：

```lang-swift
convenience <span class="hljs-keyword">init</span>(parameters) {
    statements
}

```

### 指定构造器和便利构造器实战 {#%E6%8C%87%E5%AE%9A%E6%9E%84%E9%80%A0%E5%99%A8%E5%92%8C%E4%BE%BF%E5%88%A9%E6%9E%84%E9%80%A0%E5%99%A8%E5%AE%9E%E6%88%98}

接下来的例子将在实战中展示指定构造器、便利构造器和自动构造器的继承。它定义了包含三个类`Food`、`RecipeIngredient`以及`ShoppingListItem`的类层次结构，并将演示它们的构造器是如何相互作用的。

类层次中的基类是`Food`，它是一个简单的用来封装食物名字的类。`Food`类引入了一个叫做`name`的`String`类型属性，并且提供了两个构造器来创建`Food`实例：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Food</span> </span>{
    <span class="hljs-keyword">var</span> name: <span class="hljs-type">String</span>
    <span class="hljs-keyword">init</span>(name: <span class="hljs-type">String</span>) {
        <span class="hljs-keyword">self</span>.name = name
    }
    convenience <span class="hljs-keyword">init</span>() {
        <span class="hljs-keyword">self</span>.<span class="hljs-keyword">init</span>(name: <span class="hljs-string">"[Unnamed]"</span>)
    }
}

```

下图中展示了`Food`的构造器链：

![Food构造器链](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/Art/initializersExample01_2x.png)

类没有提供一个默认的逐一成员构造器，所以`Food`类提供了一个接受单一参数`name`的指定构造器。这个构造器可以使用一个特定的名字来创建新的`Food`实例：

```lang-swift
<span class="hljs-keyword">let</span> namedMeat = <span class="hljs-type">Food</span>(name: <span class="hljs-string">"Bacon"</span>)
<span class="hljs-comment">// namedMeat 的名字是 "Bacon”</span>

```

`Food`类中的构造器`init(name: String)`被定义为一个指定构造器，因为它能确保所有新`Food`实例的中存储型属性都被初始化。`Food`类没有父类，所以`init(name: String)`构造器不需要调用`super.init()`来完成构造。

`Food`类同样提供了一个没有参数的便利构造器 `init()`。这个`init()`构造器为新食物提供了一个默认的占位名字，通过代理调用同一类中定义的指定构造器`init(name: String)`并给参数`name`传值`[Unnamed]`来实现：

```lang-swift
<span class="hljs-keyword">let</span> mysteryMeat = <span class="hljs-type">Food</span>()
<span class="hljs-comment">// mysteryMeat 的名字是 [Unnamed]</span>

```

类层级中的第二个类是`Food`的子类`RecipeIngredient`。`RecipeIngredient`类构建了食谱中的一味调味剂。它引入了`Int`类型的数量属性`quantity`（以及从`Food`继承过来的`name`属性），并且定义了两个构造器来创建`RecipeIngredient`实例：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RecipeIngredient</span>: <span class="hljs-title">Food</span> </span>{
    <span class="hljs-keyword">var</span> quantity: <span class="hljs-type">Int</span>
    <span class="hljs-keyword">init</span>(name: <span class="hljs-type">String</span>, quantity: <span class="hljs-type">Int</span>) {
        <span class="hljs-keyword">self</span>.quantity = quantity
        <span class="hljs-keyword">super</span>.<span class="hljs-keyword">init</span>(name: name)
    }
    <span class="hljs-keyword">override</span> convenience <span class="hljs-keyword">init</span>(name: <span class="hljs-type">String</span>) {
        <span class="hljs-keyword">self</span>.<span class="hljs-keyword">init</span>(name: name, quantity: <span class="hljs-number">1</span>)
    }
}

```

下图中展示了`RecipeIngredient`类的构造器链：

![RecipeIngredient构造器](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/Art/initializersExample02_2x.png)

`RecipeIngredient`类拥有一个指定构造器`init(name: String, quantity: Int)`，它可以用来产生新`RecipeIngredient`实例的所有属性值。这个构造器一开始先将传入的`quantity`参数赋值给`quantity`属性，这个属性也是唯一在`RecipeIngredient`中新引入的属性。随后，构造器将任务向上代理给父类`Food`的`init(name: String)`。这个过程满足[两段式构造过程][14]中的安全检查1。

`RecipeIngredient`也定义了一个便利构造器`init(name: String)`，它只通过`name`来创建`RecipeIngredient`的实例。这个便利构造器假设任意`RecipeIngredient`实例的`quantity`为1，所以不需要显示指明数量即可创建出实例。这个便利构造器的定义可以让创建实例更加方便和快捷，并且避免了使用重复的代码来创建多个`quantity`为 1 的`RecipeIngredient`实例。这个便利构造器只是简单的将任务代理给了同一类里提供的指定构造器。

注意，`RecipeIngredient`的便利构造器`init(name: String)`使用了跟`Food`中指定构造器`init(name: String)`相同的参数。因为这个便利构造器重写要父类的指定构造器`init(name: String)`，必须在前面使用使用`override`标识。

在这个例子中，`RecipeIngredient`的父类是`Food`，它有一个便利构造器`init()`。这个构造器因此也被`RecipeIngredient`继承。这个继承的`init()`函数版本跟`Food`提供的版本是一样的，除了它是将任务代理给`RecipeIngredient`版本的`init(name: String)`而不是`Food`提供的版本。

所有的这三种构造器都可以用来创建新的`RecipeIngredient`实例：

```lang-swift
<span class="hljs-keyword">let</span> oneMysteryItem = <span class="hljs-type">RecipeIngredient</span>()
<span class="hljs-keyword">let</span> oneBacon = <span class="hljs-type">RecipeIngredient</span>(name: <span class="hljs-string">"Bacon"</span>)
<span class="hljs-keyword">let</span> sixEggs = <span class="hljs-type">RecipeIngredient</span>(name: <span class="hljs-string">"Eggs"</span>, quantity: <span class="hljs-number">6</span>)

```

类层级中第三个也是最后一个类是`RecipeIngredient`的子类，叫做`ShoppingListItem`。这个类构建了购物单中出现的某一种调味料。

购物单中的每一项总是从`unpurchased`未购买状态开始的。为了展现这一事实，`ShoppingListItem`引入了一个布尔类型的属性`purchased`，它的默认值是`false`。`ShoppingListItem`还添加了一个计算型属性`description`，它提供了关于`ShoppingListItem`实例的一些文字描述：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ShoppingListItem</span>: <span class="hljs-title">RecipeIngredient</span> </span>{
    <span class="hljs-keyword">var</span> purchased = <span class="hljs-built_in">false</span>
    <span class="hljs-keyword">var</span> description: <span class="hljs-type">String</span> {
    <span class="hljs-keyword">var</span> output = <span class="hljs-string">"<span class="hljs-subst">\(quantity)</span> x <span class="hljs-subst">\(name.lowercaseString)</span>"</span>
        output += purchased ? <span class="hljs-string">" ✔"</span> : <span class="hljs-string">" ✘"</span>
        <span class="hljs-keyword">return</span> output
    }
}

```

> 注意：
> `ShoppingListItem`没有定义构造器来为`purchased`提供初始化值，这是因为任何添加到购物单的项的初始状态总是未购买。

由于它为自己引入的所有属性都提供了默认值，并且自己没有定义任何构造器，`ShoppingListItem`将自动继承所有父类中的指定构造器和便利构造器。

下图种展示了所有三个类的构造器链：

![三类构造器图](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/Art/initializersExample03_2x.png)

你可以使用全部三个继承来的构造器来创建`ShoppingListItem`的新实例：

```lang-swift
<span class="hljs-keyword">var</span> breakfastList = [
    <span class="hljs-type">ShoppingListItem</span>(),
    <span class="hljs-type">ShoppingListItem</span>(name: <span class="hljs-string">"Bacon"</span>),
    <span class="hljs-type">ShoppingListItem</span>(name: <span class="hljs-string">"Eggs"</span>, quantity: <span class="hljs-number">6</span>),
]
breakfastList[<span class="hljs-number">0</span>].name = <span class="hljs-string">"Orange juice"</span>
breakfastList[<span class="hljs-number">0</span>].purchased = <span class="hljs-built_in">true</span>
<span class="hljs-keyword">for</span> item <span class="hljs-keyword">in</span> breakfastList {
    <span class="hljs-built_in">println</span>(item.description)
}
<span class="hljs-comment">// 1 x orange juice ✔</span>
<span class="hljs-comment">// 1 x bacon ✘</span>
<span class="hljs-comment">// 6 x eggs ✘</span>

```

如上所述，例子中通过字面量方式创建了一个新数组`breakfastList`，它包含了三个新的`ShoppingListItem`实例，因此数组的类型也能自动推导为`ShoppingListItem[]`。在数组创建完之后，数组中第一个`ShoppingListItem`实例的名字从`[Unnamed]`修改为`Orange juice`，并标记为已购买。接下来通过遍历数组每个元素并打印它们的描述值，展示了所有项当前的默认状态都已按照预期完成了赋值。

## 可失败构造器 {#%E5%8F%AF%E5%A4%B1%E8%B4%A5%E6%9E%84%E9%80%A0%E5%99%A8}

如果一个类，结构体或枚举类型的对象，在构造自身的过程中有可能失败，则为其定义一个可失败构造器，是非常有必要的。这里所指的“失败”是指，如给构造器传入无效的参数值，或缺少某种所需的外部资源，又或是不满足某种必要的条件等。

为了妥善处理这种构造过程中可能会失败的情况。你可以在一个类，结构体或是枚举类型的定义中，添加一个或多个可失败构造器。其语法为在`init`关键字后面加添问号`(init?)`。

> 注意：
>
> 可失败构造器的参数名和参数类型，不能与其它非可失败构造器的参数名，及其类型相同。

可失败构造器，在构建对象的过程中，创建一个其自身类型为可选类型的对象。你通过`return nil` 语句，来表明可失败构造器在何种情况下“失败”。

> 注意：
>
> 严格来说，构造器都不支持返回值。因为构造器本身的作用，只是为了能确保对象自身能被正确构建。所以即使你在表明可失败构造器，失败的这种情况下，用到了`return nil`。也不要在表明可失败构造器成功的这种情况下，使用关键字 `return`。

下例中，定义了一个名为`Animal`的结构体，其中有一个名为`species`的，`String`类型的常量属性。同时该结构体还定义了一个，带一个`String`类型参数`species`的,可失败构造器。这个可失败构造器，被用来检查传入的参数是否为一个空字符串，如果为空字符串，则该可失败构造器，构建对象失败，否则成功。

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">Animal</span> </span>{
    <span class="hljs-keyword">let</span> species: <span class="hljs-type">String</span>
    <span class="hljs-keyword">init</span>?(species: <span class="hljs-type">String</span>) {
        <span class="hljs-keyword">if</span> species.isEmpty { <span class="hljs-keyword">return</span> <span class="hljs-built_in">nil</span> }
        <span class="hljs-keyword">self</span>.species = species
    }
}

```

你可以通过该可失败构造器来构建一个Animal的对象，并检查其构建过程是否成功。

```lang-swift
<span class="hljs-keyword">let</span> someCreature = <span class="hljs-type">Animal</span>(species: <span class="hljs-string">"Giraffe"</span>)
<span class="hljs-comment">// someCreature 的类型是 Animal? 而不是 Animal</span>

<span class="hljs-keyword">if</span> <span class="hljs-keyword">let</span> giraffe = someCreature {
    <span class="hljs-built_in">println</span>(<span class="hljs-string">"An animal was initialized with a species of <span class="hljs-subst">\(giraffe.species)</span>"</span>)
}
<span class="hljs-comment">// 打印 "An animal was initialized with a species of Giraffe"</span>

```

如果你给该可失败构造器传入一个空字符串作为其参数，则该可失败构造器失败。

```lang-swift
<span class="hljs-keyword">let</span> anonymousCreature = <span class="hljs-type">Animal</span>(species: <span class="hljs-string">""</span>)
<span class="hljs-comment">// anonymousCreature 的类型是 Animal?, 而不是 Animal</span>

<span class="hljs-keyword">if</span> anonymousCreature == <span class="hljs-built_in">nil</span> {
    <span class="hljs-built_in">println</span>(<span class="hljs-string">"The anonymous creature could not be initialized"</span>)
}
<span class="hljs-comment">// 打印 "The anonymous creature could not be initialized"</span>

```

> 注意：
>
> 空字符串（`""`）和一个值为`nil`的可选类型的字符串是两个完全不同的概念。上例中的空字符串（`""`）其实是一个有效的，非可选类型的字符串。这里我们只所以让`Animal`的可失败构造器，构建对象失败，只是因为对于`Animal`这个类的`species`属性来说，它更适合有一个具体的值，而不是空字符串。

### 枚举类型的可失败构造器 {#%E6%9E%9A%E4%B8%BE%E7%B1%BB%E5%9E%8B%E7%9A%84%E5%8F%AF%E5%A4%B1%E8%B4%A5%E6%9E%84%E9%80%A0%E5%99%A8}

你可以通过构造一个带一个或多个参数的可失败构造器来获取枚举类型中特定的枚举成员。还能在参数不满足你所期望的条件时，导致构造失败。

下例中，定义了一个名为TemperatureUnit的枚举类型。其中包含了三个可能的枚举成员(`Kelvin`，`Celsius`，和 `Fahrenheit`)和一个被用来找到`Character`值所对应的枚举成员的可失败构造器：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">enum</span> <span class="hljs-title">TemperatureUnit</span> </span>{
    <span class="hljs-keyword">case</span> <span class="hljs-type">Kelvin</span>, <span class="hljs-type">Celsius</span>, <span class="hljs-type">Fahrenheit</span>
    <span class="hljs-keyword">init</span>?(symbol: <span class="hljs-type">Character</span>) {
        <span class="hljs-keyword">switch</span> symbol {
        <span class="hljs-keyword">case</span> <span class="hljs-string">"K"</span>:
            <span class="hljs-keyword">self</span> = .<span class="hljs-type">Kelvin</span>
        <span class="hljs-keyword">case</span> <span class="hljs-string">"C"</span>:
            <span class="hljs-keyword">self</span> = .<span class="hljs-type">Celsius</span>
        <span class="hljs-keyword">case</span> <span class="hljs-string">"F"</span>:
            <span class="hljs-keyword">self</span> = .<span class="hljs-type">Fahrenheit</span>
        <span class="hljs-keyword">default</span>:
            <span class="hljs-keyword">return</span> <span class="hljs-built_in">nil</span>
        }
    }
}

```

你可以通过给该可失败构造器传递合适的参数来获取这三个枚举成员中相匹配的其中一个枚举成员。当参数的值不能与任意一枚举成员相匹配时，该枚举类型的构建过程失败：

```lang-swift
<span class="hljs-keyword">let</span> fahrenheitUnit = <span class="hljs-type">TemperatureUnit</span>(symbol: <span class="hljs-string">"F"</span>)
<span class="hljs-keyword">if</span> fahrenheitUnit != <span class="hljs-built_in">nil</span> {
    <span class="hljs-built_in">println</span>(<span class="hljs-string">"This is a defined temperature unit, so initialization succeeded."</span>)
}
<span class="hljs-comment">// 打印 "This is a defined temperature unit, so initialization succeeded."</span>

<span class="hljs-keyword">let</span> unknownUnit = <span class="hljs-type">TemperatureUnit</span>(symbol: <span class="hljs-string">"X"</span>)
<span class="hljs-keyword">if</span> unknownUnit == <span class="hljs-built_in">nil</span> {
    <span class="hljs-built_in">println</span>(<span class="hljs-string">"This is not a defined temperature unit, so initialization failed."</span>)
}
<span class="hljs-comment">// 打印 "This is not a defined temperature unit, so initialization failed."</span>

```

### 带原始值的枚举类型的可失败构造器 {#%E5%B8%A6%E5%8E%9F%E5%A7%8B%E5%80%BC%E7%9A%84%E6%9E%9A%E4%B8%BE%E7%B1%BB%E5%9E%8B%E7%9A%84%E5%8F%AF%E5%A4%B1%E8%B4%A5%E6%9E%84%E9%80%A0%E5%99%A8}

带原始值的枚举类型会自带一个可失败构造器`init?(rawValue:)`,该可失败构造器有一个名为`rawValue`的默认参数,其类型和枚举类型的原始值类型一致，如果该参数的值能够和枚举类型成员所带的原始值匹配，则该构造器构造一个带此原始值的枚举成员，否则构造失败。

因此上面的 TemperatureUnit的例子可以重写为：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">enum</span> <span class="hljs-title">TemperatureUnit</span>: <span class="hljs-title">Character</span> </span>{
    <span class="hljs-keyword">case</span> <span class="hljs-type">Kelvin</span> = <span class="hljs-string">"K"</span>, <span class="hljs-type">Celsius</span> = <span class="hljs-string">"C"</span>, <span class="hljs-type">Fahrenheit</span> = <span class="hljs-string">"F"</span>
}

<span class="hljs-keyword">let</span> fahrenheitUnit = <span class="hljs-type">TemperatureUnit</span>(rawValue: <span class="hljs-string">"F"</span>)
<span class="hljs-keyword">if</span> fahrenheitUnit != <span class="hljs-built_in">nil</span> {
    <span class="hljs-built_in">println</span>(<span class="hljs-string">"This is a defined temperature unit, so initialization succeeded."</span>)
}
<span class="hljs-comment">// prints "This is a defined temperature unit, so initialization succeeded."</span>

<span class="hljs-keyword">let</span> unknownUnit = <span class="hljs-type">TemperatureUnit</span>(rawValue: <span class="hljs-string">"X"</span>)
<span class="hljs-keyword">if</span> unknownUnit == <span class="hljs-built_in">nil</span> {
    <span class="hljs-built_in">println</span>(<span class="hljs-string">"This is not a defined temperature unit, so initialization failed."</span>)
}
<span class="hljs-comment">// prints "This is not a defined temperature unit, so initialization failed."</span>

```

### 类的可失败构造器 {#%E7%B1%BB%E7%9A%84%E5%8F%AF%E5%A4%B1%E8%B4%A5%E6%9E%84%E9%80%A0%E5%99%A8}

值类型（如结构体或枚举类型）的可失败构造器，对何时何地触发构造失败这个行为没有任何的限制。比如在前面的例子中，结构体`Animal`的可失败构造器触发失败的行为，甚至发生在`species`属性的值被初始化以前。而对类而言，就没有那么幸运了。类的可失败构造器只能在所有的类属性被初始化后和所有类之间的构造器之间的代理调用发生完后触发失败行为。

下例子中，定义了一个名为`Product`的类，其内部结构和结构体`Animal`很相似，内部也有一个名为`name`的`String`类型的属性。由于该属性的值同样不能为空字符串，所以我们加入了可失败构造器来确保该类满足上述条件。但由于`Product`类不是一个结构体，所以当想要在该类中添加可失败构造器触发失败条件时，必须确保`name`属性被初始化。因此我们把`name`属性的`String`类型做了一点点小小的修改，把其改为隐式解析可选类型（`String!`），来确保可失败构造器触发失败条件时，所有类属性都被初始化了。因为所有可选类型都有一个默认的初始值`nil`。因此最后`Product`类可写为：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Product</span> </span>{
    <span class="hljs-keyword">let</span> name: <span class="hljs-type">String</span>!
    <span class="hljs-keyword">init</span>?(name: <span class="hljs-type">String</span>) {
        <span class="hljs-keyword">if</span> name.isEmpty { <span class="hljs-keyword">return</span> <span class="hljs-built_in">nil</span> }
        <span class="hljs-keyword">self</span>.name = name
    }
}

```

因为`name`属性是一个常量，所以一旦`Product`类构造成功，`name`属性肯定有一个非`nil`的值。因此完全可以放心大胆的直接访问`Product`类的`name`属性，而不用考虑去检查`name`属性是否有值。

```lang-swift
<span class="hljs-keyword">if</span> <span class="hljs-keyword">let</span> bowTie = <span class="hljs-type">Product</span>(name: <span class="hljs-string">"bow tie"</span>) {
    <span class="hljs-comment">// 不需要检查 bowTie.name == nil</span>
    <span class="hljs-built_in">println</span>(<span class="hljs-string">"The product's name is <span class="hljs-subst">\(bowTie.name)</span>"</span>)
}
<span class="hljs-comment">// 打印 "The product's name is bow tie"</span>

```

### 构造失败的传递 {#%E6%9E%84%E9%80%A0%E5%A4%B1%E8%B4%A5%E7%9A%84%E4%BC%A0%E9%80%92}

可失败构造器同样满足在[构造器链][13]中所描述的构造规则。其允许在同一类，结构体和枚举中横向代理其他的可失败构造器。类似的，子类的可失败构造器也能向上代理基类的可失败构造器。

无论是向上代理还是横向代理，如果你代理的可失败构造器，在构造过程中触发了构造失败的行为，整个构造过程都将被立即终止，接下来任何的构造代码都将不会被执行。

> 注意：
>
> 可失败构造器也可以代理调用其它的非可失败构造器。通过这个方法，你可以为已有的构造过程加入构造失败的条件。

下面这个例子，定义了一个名为`CartItem`的`Product`类的子类。这个类建立了一个在线购物车中的物品的模型，它有一个名为`quantity`的常量参数，用来表示该物品的数量至少为1：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CartItem</span>: <span class="hljs-title">Product</span> </span>{
    <span class="hljs-keyword">let</span> quantity: <span class="hljs-type">Int</span>!
    <span class="hljs-keyword">init</span>?(name: <span class="hljs-type">String</span>, quantity: <span class="hljs-type">Int</span>) {
        <span class="hljs-keyword">super</span>.<span class="hljs-keyword">init</span>(name: name)
        <span class="hljs-keyword">if</span> quantity < <span class="hljs-number">1</span> { <span class="hljs-keyword">return</span> <span class="hljs-built_in">nil</span> }
        <span class="hljs-keyword">self</span>.quantity = quantity
    }
}

```

和`Product`类中的`name`属性相类似的，`CartItem`类中的`quantity`属性的类型也是一个隐式解析可选类型，只不过由（`String！`）变为了（`Int!`）。这样做都是为了确保在构造过程中，该属性在被赋予特定的值之前能有一个默认的初始值nil。

可失败构造器总是先向上代理调用基类,`Product`的构造器 `init(name:)`。这满足了可失败构造器在触发构造失败这个行为前必须总是执行构造代理调用这个条件。

如果由于`name`的值为空而导致基类的构造器在构造过程中失败。则整个`CartIem`类的构造过程都将失败，后面的子类的构造过程都将不会被执行。如果基类构建成功，则继续运行子类的构造器代码。

如果你构造了一个`CartItem`对象，并且该对象的`name`属性不为空以及`quantity`属性为1或者更多，则构造成功：

```lang-swift
<span class="hljs-keyword">if</span> <span class="hljs-keyword">let</span> twoSocks = <span class="hljs-type">CartItem</span>(name: <span class="hljs-string">"sock"</span>, quantity: <span class="hljs-number">2</span>) {
    <span class="hljs-built_in">println</span>(<span class="hljs-string">"Item: <span class="hljs-subst">\(twoSocks.name)</span>, quantity: <span class="hljs-subst">\(twoSocks.quantity)</span>"</span>)
}
<span class="hljs-comment">// 打印 "Item: sock, quantity: 2"</span>

```

如果你构造一个`CartItem`对象，其`quantity`的值``, 则`CartItem`的可失败构造器触发构造失败的行为:

```lang-swift
<span class="hljs-keyword">if</span> <span class="hljs-keyword">let</span> zeroShirts = <span class="hljs-type">CartItem</span>(name: <span class="hljs-string">"shirt"</span>, quantity: <span class="hljs-number">0</span>) {
    <span class="hljs-built_in">println</span>(<span class="hljs-string">"Item: <span class="hljs-subst">\(zeroShirts.name)</span>, quantity: <span class="hljs-subst">\(zeroShirts.quantity)</span>"</span>)
} <span class="hljs-keyword">else</span> {
    <span class="hljs-built_in">println</span>(<span class="hljs-string">"Unable to initialize zero shirts"</span>)
}
<span class="hljs-comment">// 打印 "Unable to initialize zero shirts"</span>

```

类似的, 如果你构造一个`CartItem`对象，但其`name`的值为空, 则基类`Product`的可失败构造器将触发构造失败的行为，整个`CartItem`的构造行为同样为失败：

```lang-swift
<span class="hljs-keyword">if</span> <span class="hljs-keyword">let</span> oneUnnamed = <span class="hljs-type">CartItem</span>(name: <span class="hljs-string">""</span>, quantity: <span class="hljs-number">1</span>) {
    <span class="hljs-built_in">println</span>(<span class="hljs-string">"Item: <span class="hljs-subst">\(oneUnnamed.name)</span>, quantity: <span class="hljs-subst">\(oneUnnamed.quantity)</span>"</span>)
} <span class="hljs-keyword">else</span> {
    <span class="hljs-built_in">println</span>(<span class="hljs-string">"Unable to initialize one unnamed product"</span>)
}
<span class="hljs-comment">// 打印 "Unable to initialize one unnamed product"</span>

```

### 覆盖一个可失败构造器 {#%E8%A6%86%E7%9B%96%E4%B8%80%E4%B8%AA%E5%8F%AF%E5%A4%B1%E8%B4%A5%E6%9E%84%E9%80%A0%E5%99%A8}

就如同其它构造器一样，你也可以用子类的可失败构造器覆盖基类的可失败构造器。或者你也可以用子类的非可失败构造器覆盖一个基类的可失败构造器。这样做的好处是，即使基类的构造器为可失败构造器，但当子类的构造器在构造过程不可能失败时，我们也可以把它修改过来。

注意当你用一个子类的非可失败构造器覆盖了一个父类的可失败构造器时，子类的构造器将不再能向上代理父类的可失败构造器。一个非可失败的构造器永远也不能代理调用一个可失败构造器。

> 注意：
>
> 你可以用一个非可失败构造器覆盖一个可失败构造器，但反过来却行不通。

下例定义了一个名为`Document`的类，这个类中的`name`属性允许为`nil`和一个非空字符串，但不能是一个空字符串：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Document</span> </span>{
    <span class="hljs-keyword">var</span> name: <span class="hljs-type">String</span>?
    <span class="hljs-comment">// 该构造器构建了一个name属性值为nil的document对象</span>
    <span class="hljs-keyword">init</span>() {}
    <span class="hljs-comment">// 该构造器构建了一个name属性值为非空字符串的document对象</span>
    <span class="hljs-keyword">init</span>?(name: <span class="hljs-type">String</span>) {
        <span class="hljs-keyword">if</span> name.isEmpty { <span class="hljs-keyword">return</span> <span class="hljs-built_in">nil</span> }
        <span class="hljs-keyword">self</span>.name = name
    }
}

```

下面这个例子，定义了一个名为`AutomaticallyNamedDocument`的`Document`类的子类。这个子类覆盖了基类的两个指定构造器。确保了不论在何种情况下`name`属性总是有一个非空字符串`[Untitled]`的值。

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AutomaticallyNamedDocument</span>: <span class="hljs-title">Document</span> </span>{
    <span class="hljs-keyword">override</span> <span class="hljs-keyword">init</span>() {
        <span class="hljs-keyword">super</span>.<span class="hljs-keyword">init</span>()
        <span class="hljs-keyword">self</span>.name = <span class="hljs-string">"[Untitled]"</span>
    }
    <span class="hljs-keyword">override</span> <span class="hljs-keyword">init</span>(name: <span class="hljs-type">String</span>) {
        <span class="hljs-keyword">super</span>.<span class="hljs-keyword">init</span>()
        <span class="hljs-keyword">if</span> name.isEmpty {
            <span class="hljs-keyword">self</span>.name = <span class="hljs-string">"[Untitled]"</span>
        } <span class="hljs-keyword">else</span> {
            <span class="hljs-keyword">self</span>.name = name
        }
    }
}

```

````lang-AutomaticallyNamedDocument```用一个非可失败构造器```init(name:)```,覆盖了基类的可失败构造器```init?(name:)```。因为子类用不同的方法处理了```name```属性的值为一个空字符串的这种情况。所以子类将不再需要一个可失败的构造器。

###可失败构造器 init!

通常来说我们通过在```init```关键字后添加问号的方式来定义一个可失败构造器，但你也可以使用通过在```init```后面添加惊叹号的方式来定义一个可失败构造器```（init!）```，该可失败构造器将会构建一个特定类型的隐式解析可选类型的对象。

你可以在 ```init？```构造器中代理调用 ```init！```构造器，反之亦然。
你也可以用 ```init？```覆盖 ```init！```，反之亦然。
你还可以用 ```init```代理调用```init！```，但这会触发一个断言：是否 ```init！```构造器会触发构造失败？

<a name="required_initializers"></a>
##必要构造器

在类的构造器前添加```required```修饰符表明所有该类的子类都必须实现该构造器：

```swift
class SomeClass {
    required init() {
        // 在这里添加该必要构造器的实现代码
    }
}

````

当子类覆盖基类的必要构造器时，必须在子类的构造器前同样添加`required`修饰符以确保当其它类继承该子类时，该构造器同为必要构造器。在覆盖基类的必要构造器时，不需要添加`override`修饰符：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SomeSubclass</span>: <span class="hljs-title">SomeClass</span> </span>{
    required <span class="hljs-keyword">init</span>() {
        <span class="hljs-comment">// 在这里添加子类必要构造器的实现代码</span>
    }
}

```

> 注意：
>
> 如果子类继承的构造器能满足必要构造器的需求，则你无需显示的在子类中提供必要构造器的实现。

## 通过闭包和函数来设置属性的默认值 {#%E9%80%9A%E8%BF%87%E9%97%AD%E5%8C%85%E5%92%8C%E5%87%BD%E6%95%B0%E6%9D%A5%E8%AE%BE%E7%BD%AE%E5%B1%9E%E6%80%A7%E7%9A%84%E9%BB%98%E8%AE%A4%E5%80%BC}

如果某个存储型属性的默认值需要特别的定制或准备，你就可以使用闭包或全局函数来为其属性提供定制的默认值。每当某个属性所属的新类型实例创建时，对应的闭包或函数会被调用，而它们的返回值会当做默认值赋值给这个属性。

这种类型的闭包或函数一般会创建一个跟属性类型相同的临时变量，然后修改它的值以满足预期的初始状态，最后将这个临时变量的值作为属性的默认值进行返回。

下面列举了闭包如何提供默认值的代码概要：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SomeClass</span> </span>{
    <span class="hljs-keyword">let</span> someProperty: <span class="hljs-type">SomeType</span> = {
        <span class="hljs-comment">// 在这个闭包中给 someProperty 创建一个默认值</span>
        <span class="hljs-comment">// someValue 必须和 SomeType 类型相同</span>
        <span class="hljs-keyword">return</span> someValue
        }()
}

```

注意闭包结尾的大括号后面接了一对空的小括号。这是用来告诉 Swift 需要立刻执行此闭包。如果你忽略了这对括号，相当于是将闭包本身作为值赋值给了属性，而不是将闭包的返回值赋值给属性。

> 注意：
> 如果你使用闭包来初始化属性的值，请记住在闭包执行时，实例的其它部分都还没有初始化。这意味着你不能够在闭包里访问其它的属性，就算这个属性有默认值也不允许。同样，你也不能使用隐式的`self`属性，或者调用其它的实例方法。

下面例子中定义了一个结构体`Checkerboard`，它构建了西洋跳棋游戏的棋盘：

![西洋跳棋棋盘](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/Art/checkersBoard_2x.png)

西洋跳棋游戏在一副黑白格交替的 10×10 的棋盘中进行。为了呈现这副游戏棋盘，`Checkerboard`结构体定义了一个属性`boardColors`，它是一个包含 100 个布尔值的数组。数组中的某元素布尔值为`true`表示对应的是一个黑格，布尔值为`false`表示对应的是一个白格。数组中第一个元素代表棋盘上左上角的格子，最后一个元素代表棋盘上右下角的格子。

`boardColor`数组是通过一个闭包来初始化和组装颜色值的：

```lang-swift
<span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">Checkerboard</span> </span>{
    <span class="hljs-keyword">let</span> boardColors: [<span class="hljs-type">Bool</span>] = {
        <span class="hljs-keyword">var</span> temporaryBoard = [<span class="hljs-type">Bool</span>]()
        <span class="hljs-keyword">var</span> isBlack = <span class="hljs-built_in">false</span>
        <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> <span class="hljs-number">1</span>...<span class="hljs-number">10</span> {
            <span class="hljs-keyword">for</span> j <span class="hljs-keyword">in</span> <span class="hljs-number">1</span>...<span class="hljs-number">10</span> {
                temporaryBoard.append(isBlack)
                isBlack = !isBlack
            }
            isBlack = !isBlack
        }
        <span class="hljs-keyword">return</span> temporaryBoard
        }()
    <span class="hljs-func"><span class="hljs-keyword">func</span> <span class="hljs-title">squareIsBlackAtRow</span><span class="hljs-params">(row: Int, column: Int)</span> -> <span class="hljs-title">Bool</span> </span>{
        <span class="hljs-keyword">return</span> boardColors[(row * <span class="hljs-number">10</span>) + column]
    }
}

```

每当一个新的`Checkerboard`实例创建时，对应的赋值闭包会执行，一系列颜色值会被计算出来作为默认值赋值给`boardColors`。上面例子中描述的闭包将计算出棋盘中每个格子合适的颜色，将这些颜色值保存到一个临时数组`temporaryBoard`中，并在构建完成时将此数组作为闭包返回值返回。这个返回的值将保存到`boardColors`中，并可以通`squareIsBlackAtRow`这个工具函数来查询。

```lang-swift
<span class="hljs-keyword">let</span> board = <span class="hljs-type">Checkerboard</span>()
<span class="hljs-built_in">println</span>(board.squareIsBlackAtRow(<span class="hljs-number">0</span>, column: <span class="hljs-number">1</span>))
<span class="hljs-comment">// 输出 "true"</span>
<span class="hljs-built_in">println</span>(board.squareIsBlackAtRow(<span class="hljs-number">9</span>, column: <span class="hljs-number">9</span>))
<span class="hljs-comment">// 输出 "false"</span>
```

 [1]: http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/14_Initialization.html#setting_initial_values_for_stored_properties
 [2]: http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/14_Initialization.html#customizing_initialization
 [3]: http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/14_Initialization.html#default_initializers
 [4]: http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/14_Initialization.html#initializer_delegation_for_value_types
 [5]: http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/14_Initialization.html#class_inheritance_and_initialization
 [6]: http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/14_Initialization.html#failable_initializers
 [7]: http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/14_Initialization.html#required_initializers
 [8]: http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/14_Initialization.html#setting_a_default_property_value_with_a_closure_or_function
 [9]: http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/15_Deinitialization.html
 [10]: http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/13_Inheritance.html
 [11]: http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/20_Extensions.html
 [12]: http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/14_Initialization.html#automatic_initializer_inheritance
 [13]: http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/14_Initialization.html#initialization_chain
 [14]: http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/chapter2/14_Initialization.html#two_phase_initialization