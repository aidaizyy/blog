title: "Swift基础入门(12)：泛型"
date: 2015-07-28 10:34:05
tags:
- swift
categories: swift
toc: true
---

本篇介绍Swift的基础知识：泛型，适合任何类型的函数和类型。

<!--more-->
**Title: [Swift基础入门(12)：泛型](https://aidaizyy.github.io/swift_12)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2015-07-28](http://aidaizyy.github.io)**

# 泛型
泛型（_generic_）类似于C++中的模板，可以写出适合任何类型的函数和类型。为函数或者类型指定了模板类型，可以传入任何类型去替代模板类型。

## 泛型函数
``` swift
func swapTwoValues<T>(inout a: T, inout b: T) {
    let temporaryA = a
    a = b
    b = temporaryA
}

var someInt = 3
var anotherInt = 107
swapTwoValues(&someInt, &anotherInt)
// someInt is now 107, and anotherInt is now 3

var someString = "hello"
var anotherString = "world"
swapTwoValues(&someString, &anotherString)
// someString is now "world", and anotherString is now "hello"
```
函数`swapTwoValues`后面接尖括号和占位符`T`替代任何类型，函数的两个参数`a`和`b`的类型都指定为`T`，代表这两个参数类型是一样的，但是没有被确定，适合任何类型。
第7-10行，传入了两个整数，可以调用函数；第12-15行，传入了两个字符串，可以调用函数。

## 泛型类型
``` swift
struct Stack<T> {
    var items = [T]()
    mutating func push(item: T) {
        items.append(item)
    }
    mutating func pop() -> T {
        return items.removeLast()
    }
}

var stackOfStrings = Stack<String>()
stackOfStrings.push("uno")
stackOfStrings.push("dos")
stackOfStrings.push("tres")
stackOfStrings.push("cuatro")
// 现在栈已经有4个string了
```
结构体`Stack`被定义为泛型类型，传入参数T表示类型，比如`Stack<String>`。

## 类型约束
类型参数可以有多个，在尖括号中用逗号隔开。
类型参数的命名可以自由命名，以大写字母开头。
类型参数也可以定义类型约束。
``` swift
func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
    // function body goes here
}
```
比如上面代码中，类型`T`必须是类`SomeClass`的子类，类型`U`必须遵循协议`SomeProtocol`。
再来一个例子。
``` swift
func findIndex<T: Equatable>(array: T[], valueToFind: T) -> Int? {
    for (index, value) in enumerate(array) {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```
`Equatable`是Swift自带的协议，表示可以用`==`和`!=`进行比较，所有的标准类型都支持这个协议。因为后面的函数体中出现了`==`比较，所以必须要求类型`T`遵循`Equatable`类型。

## 关联类型
关联类型（_associated type_）声明在协议中，表示一个类型，但协议被实现前不需要指定具体类型。
下面的例子声明了一个协议`Container`，表示容器，并定义了`append`方法，`count`属性和下标。`append`方法需要传入一个参数，为了使协议能支持任何类型，传入的参数类型不确定，因此用关联类型`ItemType`替代，使用关键字`typealias`，这里不是别名的意思。
``` swift
protocol Container {
    typealias ItemType
    mutating func append(item: ItemType)
    var count: Int { get }
    subscript(i: Int) -> ItemType { get }
}
```
声明协议之后，实现这个协议。
把关联类型声明成`Int`：
``` swift
struct IntStack: Container {
    // IntStack的原始实现
    var items = [Int]()
    // 遵循Container协议的实现
    typealias ItemType = Int
    mutating func append(item: Int) {
        self.push(item)
    }
    ……
}
```
也可以把关联类型声明成泛型类型：
``` swift
struct Stack<T>: Container {
    // original Stack<T> implementation
    var items = [T]()
    // conformance to the Container protocol
    mutating func append(item: T) {
        self.push(item)
    }
}
```
这里没有定义`typealias ItemType`的类型，因为通过`append()`传入的参数就可以判断`ItemType`的类型，可以省略定义。

- 参数约束
通常对关联类型定义约束，使用`where`语句定义参数的约束，紧跟在类型参数列表后面。
下面一个例子。
``` swift
func allItemsMatch<
    C1: Container, C2: Container
    where C1.ItemType == C2.ItemType, C1.ItemType: Equatable>
    (someContainer: C1, anotherContainer: C2) -> Bool {

        // 检查两个Container的元素个数是否相同
        if someContainer.count != anotherContainer.count {
            return false
        }

        // 检查两个Container相应位置的元素彼此是否相等
        for i in 0..<someContainer.count {
            if someContainer[i] != anotherContainer[i] {
                return false
            }
        }

        // 如果所有元素检查都相同则返回true
        return true

}
```
两个类型参数`C1`和`C2`都遵循协议`Container`，紧跟`where`语句，表示`C1`的参数和`C2`的参数必须是同一个类型，且可以使用`==`或者`=!`符号。
``` swift
var stackOfStrings = Stack<String>()
stackOfStrings.push("uno")
stackOfStrings.push("dos")
stackOfStrings.push("tres")

var arrayOfStrings = ["uno", "dos", "tres"]

if allItemsMatch(stackOfStrings, arrayOfStrings) {
    println("All items match.")
} else {
    println("Not all items match.")
}
// 输出 "All items match."
```
这个函数的作用是比较两个容器的元素是否完全一样。
