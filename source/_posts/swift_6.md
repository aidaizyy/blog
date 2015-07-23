title: "Swift基础入门(6)：枚举，类和结构体"
date: 2015-07-19 22:47:43
tags:
- swift
categories: swift 
toc: true
---

本篇介绍Swift的基础知识：枚举，类和结构体的基本概念和语法。

<!--more-->
**Title: [Swift基础入门(6)：枚举，类和结构体](https://aidaizyy.github.io/swift_6)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2015-07-22](http://aidaizyy.github.io)**

# 枚举

## 枚举语法
枚举定义了一个通用来兴的一组相关的值。
``` swift
enum CompassPoint {
    case North, South
    case East
    case West
}
```
`enum`关键字把枚举的整个定义放在大括号中，`CompassPoint`是它的名称。`case`表明新的一行成员值被定义，同一行中可以定义多个成员值，用`,`隔开，这里的成员值为`North`，`South`，`East`，`West`。
``` swift
var directionToHead = CompassPoint.West
directionToHead = .East
```
变量的类型经过第一次赋值确定后，再次赋值可省略枚举类型名称，这里`directionToHead`已经被确定为`CompassPoint`的成员值，再次赋值用`.East`的形式就可以了。
定义的枚举成员是没有值的，不会自动分配值。后面会介绍存储原始值，不仅可以存储整数，也可以存储浮点数字符串等其他类型。

## 成员值
枚举类型用`switch`匹配时，必须每个成员值都考虑到，否则编译无法通过，可用`default`替代其他成员值。
``` swift
directionToHead = .South
switch directionToHead {
case .North:
    println("Lots of planets have a north")
case .South:
    println("Watch out for penguins")
case .East:
    println("Where the sun rises")
case .West:
    println("Where the skies are blue")
}
// 输出 "Watch out for penguins”
```

## 相关值
枚举类型的用法比较像C语言中的联合体（_union_），可以为成员值提供其他类型的相关值，即成员值之外的自定义信息。
相关值可以是任何类型，每个成员的数据类型也可以不一样。
``` swift
enum Barcode {
  case UPCA(Int, Int, Int)
  case QRCode(String)
}
```
枚举类型`Barcode`有两个成员值，一个是`UPCA`，它的相关值是`(Int, Int, int)`，一个是`QRCode`，它的相关值是`(String)`
``` swift
var productBarcode = Barcode.UPCA(8, 85909_51226, 3)
productBarcode = .QRCode("ABCDEFGHIJKLMNOP")
```

# 类和结构体

类（_Class_）和结构体（_Struct_）的用法和其他语言类似。
主要区别在于，类允许继承，而结构体不行；类是引用传递，而结构体是值传递。

``` swift
struct Resolution {
    var width = 0
    var height = 0
}
class VideoMode {
    var resolution = Resolution()
    var interlaced = false
    var frameRate = 0.0
    var name: String?
}

var someResolution = Resolution(width: 1920, height: 1080)
let someVideoMode = VideoMode()
someVideoMode.resolution = someResolution;
someVideoMode.interlaced = true;
someVideoMode.name = "1080i"
someVideoMode.frameRate = 25.0

var otherResolution = someResolution;
let otherVideoMode = someVideoMode;

someResolution.width = 2048
println("someResolution is now  \(someResolution.width) pixels wide")
// 输出 "someResolution is now 2048 pixels wide"
println("otherResolution is now  \(otherResolution.width) pixels wide")
// 输出 "otherResolution is now 1920 pixels wide"

someVideoMode.resolution.width = 1280
println("The width of someVideoMode is now \(someVideoMode.resolution.width)")
// 输出 "The width of someVideoMode is now 1280"
println("The width of otherVideoMode is now \(otherVideoMode.resolution.width)")
// 输出 "The width of otherVideoMode is now 1280"

if someVideoMode === otherVideoMode {
    println("someVideoMode and otherVideoMode refer to the same VideoMode instance.")
}
//输出 "someVideoMode and otherVideoMode refer to the same VideoMode instance."
```
第1-10行是类和结构体的定义，分别用`class`和`struct`表示。

第12-17行是给类和结构体创建实例，并赋值。
结构体可以在构造时逐一初始化成员，`(width: 1920, height: 1080)`，而类不可以。

第19-20行，分别用类和结构体的实例去赋值变量或常量。

第22-26行，变量`otherResolution`被结构体`someResolution`赋值时采用的是值传递，因此相互是独立的，只是成员值一样。
改变了`someResolutin`的属性`width`的值后，`otherResolution`并未受到影响。

第28-32行，常量`otherVideoMode`被类`otherVideoMode`赋值时采用的是引用传递，指向的是同一个对象。
改变了`someVideoMode`的属性`reoulution.width`的值后，`otherVideoMode`的相应属性也随之变化。
`someVideoMode`和`otherVideoMode`被声明为常量,也可以改变其中的成员属性：
因为他们都不存储实例，只存储了引用对象，没有改变引用对象，只改变了被引用的基础`VideoMode`的成员属性。
Swift中，几乎所有的基本类型，包括字符串，数组和字典等都是值传递。

第34-37行，因为两者指向同一对象，不仅仅是成员值相等的关系了，`==`等于符号并不足以描述这样的关系。
`====`恒等运算符用来形容两者指向同一对象，表示两个实例等价。

枚举，类和结构体的其他特性，参见：
[Swift基础入门(7)：属性，方法和下标脚注](http://aidaizyy.github.io/swift_7)
[Swift基础入门(8)：继承，构造过程和析构过程](http://aidaizyy.github.io/swift_8)
