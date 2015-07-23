title: "Swift基础入门(4)：条件与循环语句"
date: 2015-07-16 14:36:11
tags:
- swift
categories: swift 
toc: true
---

本篇介绍Swift的基础知识：条件语句和循环语句。

<!--more-->
**Title: [Swift基础入门(4)：条件与循环语句](https://aidaizyy.github.io/swift_4)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2015-07-16](http://aidaizyy.github.io)**

# 循环语句

## for-in循环
``` swift
for index in 1...5 {
	println(index)
}
//1
//2
//3
//4
//5
```
`for-in`循环可用于区间。
``` swift
var res = 1
for _ in 1...5 {
	res *= 2
	println(res)
}
//2
//4
//8
//16
//32
```
如果不需要区间中每项的值，可以用`_`替代。
`for-in`用于字符串，请见[Swift基础入门(2)](http://aidaizyy.github.io/swift_2)。
`for-in`用于数组集合字典，请见[Swift基础入门(3)](http://aidaizyy.github.io/swift_3)。

## for循环
`for`循环和C语言一致，格式为`for initialization; condition; increment { statements }`，区别在于没有括号。

## while循环
`while`循环和C语言一致，格式为`while condition { statements }`，区别在于没有括号。

## do-while循环
`do-while`循环和C语言一致，格式为`do { statements } while condition`，区别在于没有括号。

# 条件语句

## if语句
`if`语句和C语言一直，格式为`if condition { statements } else if condition { statements } else { statesments },区别在于没有括号，`else if`和`else`不是必须存在。

## switch语句
``` swift
let count = 300
var naturalCount: String
switch count {
case 0:
    naturalCount = "no"
case 1...3:
    naturalCount = "a few"
case 4...9:
    naturalCount = "several"
case 10...99:
    naturalCount = "tens of"
case 100...999:
    naturalCount = "hundreds of"
case 1000...999_999:
    naturalCount = "thousands of"
default:
    naturalCount = "millions and millions of"
}
println("There are \(naturalCount) stars in the Milky Way.")
// 输出 "There are hundreds of stars in the Milk Way."
```
在C语言中，通常使用`break`，避免执行了一个`case`语句后继续执行下一个`case`语句。在Swift语言中不需要添加`break`，`switch`语句只执行最前面一个符合条件的`case`语句。
`case`语句可以接类似于`1...3`的区间。
`case`语句可以接多个情况，用逗号隔开，`switch value { case value1, value2: statements }`

## 元组
``` swift
let somePoint = (1, 1)
switch somePoint {
case (0, 0):
    println("(0, 0) is at the origin")
case (_, 0):
    println("(\(somePoint.0), 0) is on the x-axis")
case (0, _):
    println("(0, \(somePoint.1)) is on the y-axis")
case (-2...2, -2...2):
    println("(\(somePoint.0), \(somePoint.1)) is inside the box")
default:
    println("(\(somePoint.0), \(somePoint.1)) is outside of the box")
}
// 输出 "(1, 1) is inside the box"
``` 
元组也可以用来判断条件，`_`用来匹配所有可能的值，也就是需要忽略的值。

## 值绑定
``` swift
let anotherPoint = (2, 0)
switch anotherPoint {
case (let x, 0):
    println("on the x-axis with an x value of \(x)")
case (0, let y):
    println("on the y-axis with a y value of \(y)")
case let (x, y):
    println("somewhere else at (\(x), \(y))")
}
```
`case`语句中，可以用临时的常量变量去绑定值并使用。

## 额外条件（Where语句）
``` swift
let yetAnotherPoint = (1, -1)
switch yetAnotherPoint {
case let (x, y) where x == y:
    println("(\(x), \(y)) is on the line x == y")
case let (x, y) where x == -y:
    println("(\(x), \(y)) is on the line x == -y")
case let (x, y):
    println("(\(x), \(y)) is just some arbitrary point")
}
// 输出 "(1, -1) is on the line x == -y"
```
`case`语句中可以使用`where`语句跟在条件后作为额外的补充条件，需要同时满足两个条件才可以执行。

# 控制转移语句
Swift一共有四种控制转移语句：
-continue
-break
-fallthrough
-return

`continue`，`break`和`return`用法和C语言基本一致。
在`switch`语句中，`continue`和`break`都针对整个`switch`语句，而不是C语言中的一个`case`语句。遇到`break`后直接退出整个`switch`语句，而不是判断下一个`case`，`continue`同理。

## 贯穿语句（Fallthrough语句）
``` swift
let integerToDescribe = 5
var description = "The number \(integerToDescribe) is"
switch integerToDescribe {
case 2, 3, 5, 7, 11, 13, 17, 19:
    description += " a prime number, and also"
    fallthrough
default:
    description += " an integer."
}
println(description)
// 输出 "The number 5 is a prime number, and also an integer."
```
Swift语言不支持在`switch`语句中贯穿多个`case`语句的情况，但有时我们需要这么做。这时我们可以加上关键字`fallthrough`，当遇到`fallthrough`时，就会继续执行下一个`case`语句。
注意：遇到`fallthrough`时会直接**执行**下一个`case`语句，而不是去**判断**条件。

## 精确控制转移
Swift语言中可以让`break`和`continue`精确地表示针对哪一个循环或条件语句，这称为带标签的语句（_Labeled Statements_）。
`label name: while condition { statements }`，之后再执行`break label name`或`continue label name`。 
``` swift
//求第一个质数
loop: for integer in 1...10 {
    switch integer {
    case 2, 3, 5, 7, 11, 13, 17, 19:
        println("\(integer) is a prime number")
        break loop
    default:
        println("\(integer) is not a prime number")
    }
}
```
上面代码中，给`for-in`循环指定了标签`loop`，我们要求得到第一个质数，所以当遇到质数后用`break loop`结束循环。如果不加标签，`break`只能结束`switch`语句，会继续执行循环，不能达到目的。
