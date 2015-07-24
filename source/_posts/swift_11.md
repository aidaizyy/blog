title: "Swift基础入门(11)：扩展和协议"
date: 2015-07-24 16:28:22
tags:
- swift
categories: swift
toc: true
---

本篇介绍Swift的基础知识：扩展（_extensions_）和协议（_protocol_）的语法和实例。

<!--more-->
**Title: [Swift基础入门(11)：扩展和协议](https://aidaizyy.github.io/swift_11)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2015-07-24](http://aidaizyy.github.io)**

# 扩展
扩展可以为枚举，类和结构体：
- 添加计算型实例属性和计算型类型属性
- 添加实例方法和类型方法
- 添加构造器
- 添加下标
- 添加新的嵌套类型
- 已有类型适配协议

## 语法
声明扩展使用关键字`extension`：
``` swfit
extension SomeType {
    // 新功能
}

extension SomeType: SomeProtocol, AnotherProtocol {
    // 已有类型适配的协议实现
}
```

## 计算型属性
扩展只能添加计算型属性，包括实例属性和类型属性，但是不能添加存储属性和属性观察器。
下面的例子为`Double`类添加了5个计算型实例属性，因为都是只读属性，所以省略了`get`关键字。
``` swift
extension Double {
    var km: Double { return self * 1_000.0 }
    var m : Double { return self }
    var cm: Double { return self / 100.0 }
    var mm: Double { return self / 1_000.0 }
    var ft: Double { return self / 3.28084 }
}
let oneInch = 25.4.mm
println("One inch is \(oneInch) meters")
// 打印输出："One inch is 0.0254 meters"
let threeFeet = 3.ft
println("Three feet is \(threeFeet) meters")
// 打印输出："Three feet is 0.914399970739201 meters"
```

## 方法
下面的例子为`Int`类型添加了1个实例方法，实现了多次执行某任务的功能。
``` swift
extension Int {
    func repetitions(task: () -> ()) {
        for i in 0..<self {
            task()
}}}

3.repetitions({
    println("Hello!")
    })
// Hello!
// Hello!
// Hello!
```
这个实例方法传入一个无参数无返回值的函数，没有返回值。

扩展的方法可以修改实例本身，使用`mutating`关键字。
下面的例子添加了一个实现平方计算的方法。
``` swift
extension Int {
    mutating func square() {
        self = self * self
    }
}
var someInt = 3
someInt.square()
// someInt 现在值是 9
```
枚举和结构体修改`self`或者属性的方法都必须标注为`mutating`。

## 下标
下面的例子为`Int`类型添加了1个下标，返回整数的从右数第`index`位上的个位数。
``` swift
extension Int {
    subscript(var digitIndex: Int) -> Int {
        var decimalBase = 1
            while digitIndex > 0 {
                decimalBase *= 10
                --digitIndex
            }
            return (self / decimalBase) % 10
    }
}
746381295[0]
// returns 5
746381295[1]
// returns 9
746381295[2]
// returns 2
746381295[8]
// returns 7
```

## 构造器
扩展只能添加便利构造器，但是不能添加指定构造器和析构函数。
``` swift
struct Size {
    var width = 0.0, height = 0.0
}
struct Point {
    var x = 0.0, y = 0.0
}
struct Rect {
    var origin = Point()
    var size = Size()
}

extension Rect {
    init(center: Point, size: Size) {
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
        self.init(origin: Point(x: originX, y: originY), size: size)
    }
}

let centerRect = Rect(center: Point(x: 4.0, y: 4.0),
    size: Size(width: 3.0, height: 3.0))
// centerRect的原点是 (2.5, 2.5)，大小是 (3.0, 3.0)
```
上面的例子为类`Rect`添加了构造器，传入`Point`和`Size`参数，初始化了`Rect`。

## 嵌套类型
扩展可以向已有的枚举，类和结构体添加新的嵌套类型。
下面的例子为`Character`添加了新的枚举类型：
``` swift
extension Character {
    enum Kind {
        case Vowel, Consonant, Other
    }
    var kind: Kind {
        switch String(self).lowercaseString {
        case "a", "e", "i", "o", "u":
            return .Vowel
        case "b", "c", "d", "f", "g", "h", "j", "k", "l", "m",
             "n", "p", "q", "r", "s", "t", "v", "w", "x", "y", "z":
            return .Consonant
        default:
            return .Other
        }
    }
}

func printLetterKinds(word: String) {
    println("'\(word)' is made up of the following kinds of letters:")
    for character in word {
        switch character.kind {
        case .Vowel:
            print("vowel ")
        case .Consonant:
            print("consonant ")
        case .Other:
            print("other ")
        }
    }
    print("\n")
}
printLetterKinds("Hello")
// 'Hello' is made up of the following kinds of letters:
// consonant vowel consonant consonant vowel
```
枚举类型`Kind`表示字母是元音，辅音还是其他类型。
添加了计算型方法，返回字母对应的`Kind`枚举成员类型。

# 协议
