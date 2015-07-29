title: "Swift基础入门(13)：高级运算符"
date: 2015-07-28 11:59:40
tags:
- swift
categories: swift 
toc: true
---

本篇介绍Swift的基础知识：高级运算符，包括位运算符，溢出运算符和运算符重载与自定义。

<!--more-->
**Title: [Swift基础入门(13)：高级操作符](https://aidaizyy.github.io/swift_13)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2015-07-28](http://aidaizyy.github.io)**

# 高级操作符

## 位运算符
Swift中位运算符和C语言基本一致，不再详细介绍。
- ~：取反
- &：与
- |：或
- ^：异或
- <<：左移
- \>\>：右移
左移右移同样分逻辑移位和算术移位

## 溢出运算符
Swift默认不能溢出，如果故意要溢出必须采用溢出运算。
- &+：溢出加法
- &-：溢出减法
- &*：溢出乘法
- &/：溢出除法
- &%：溢出求余
``` swift
var willOverflow = UInt8.max
// willOverflow 等于UInt8的最大整数 255
willOverflow = willOverflow &+ 1
// 此时 willOverflow 等于 0

var willUnderflow = UInt8.min
// willUnderflow 等于UInt8的最小值0
willUnderflow = willUnderflow &- 1
// 此时 willUnderflow 等于 255

var signedUnderflow = Int8.min
// signedUnderflow 等于最小的有符整数 -128
signedUnderflow = signedUnderflow &- 1
// 此时 signedUnderflow 等于 127

let x = 1
let y = x &/ 0
// y 等于 0
```
不一样的地方在于：一个数除以0或者对0求余数，即`i / 0`或者`i % 0`，其他默认会溢出的语言会报错。但是Swift的`i &/ 0`和`i &% 0`溢出运算会让结果都等于0。

## 运算符函数
对运算符重载，让已有的运算符，如`+`，`-`等基本运算符能对自定义的类和结构体进行运算。
- @infix：中置运算符
- @prefix：前置运算符
- @postfix：后置运算符
- @assignment：组合赋值运算符
下面对`+`进行重载，函数传入了两个参数，表示双目运算符，有两个操作数。
函数前加上关键字`@infix`，表示`+`作为中置运算符，放在两个操作数的中间。
``` swift
struct Vector2D {
    var x = 0.0, y = 0.0
}
@infix func + (left: Vector2D, right: Vector2D) -> Vector2D {
    return Vector2D(x: left.x + right.x, y: left.y + right.y)
}

let vector = Vector2D(x: 3.0, y: 1.0)
let anotherVector = Vector2D(x: 2.0, y: 4.0)
let combinedVector = vector + anotherVector
// combinedVector 是一个新的Vector2D, 值为 (5.0, 5.0)
```
前置和后置运算分别加上关键字`@prefix`和`@postfix`，表示把运算符放在操作数前面或后面。
函数只传入一个参数，表示弹幕运算符，只有一个操作数。
``` swift
@prefix func - (vector: Vector2D) -> Vector2D {
    return Vector2D(x: -vector.x, y: -vector.y)
}

let positive = Vector2D(x: 3.0, y: 4.0)
let negative = -positive
// negative 为 (-3.0, -4.0)
let alsoPositive = -negative
// alsoPositive 为 (3.0, 4.0)
```
如果我们要重载`+=`，`-=`等运算符加上赋值符的组合赋值运算符，需要使用`@assignment`关键字。也可以和前置后置组合起来，形成`@prefix @assignment`或`@postfix @assignment`。
``` swift
@assignment func += (inout left: Vector2D, right: Vector2D) {
    left = left + right		//`+`在前面的例子中已经定义过了
}

var original = Vector2D(x: 1.0, y: 2.0)
let vectorToAdd = Vector2D(x: 3.0, y: 4.0)
original += vectorToAdd
// original 现在为 (4.0, 6.0)

@prefix @assignment func ++ (inout vector: Vector2D) -> Vector2D {
    vector += Vector2D(x: 1.0, y: 1.0)
    return vector
}

var toIncrement = Vector2D(x: 3.0, y: 4.0)
let afterIncrement = ++toIncrement
// toIncrement 现在是 (4.0, 5.0)
// afterIncrement 现在也是 (4.0, 5.0)
```
比较运算符重载`==`和`!=`类似于其他中置运算符。
``` swift
@infix func == (left: Vector2D, right: Vector2D) -> Bool {
    return (left.x == right.x) && (left.y == right.y)
}

@infix func != (left: Vector2D, right: Vector2D) -> Bool {
    return !(left == right)
}

let twoThree = Vector2D(x: 2.0, y: 3.0)
let anotherTwoThree = Vector2D(x: 2.0, y: 3.0)
if twoThree == anotherTwoThree {
    println("这两个向量是相等的.")
}
// prints "这两个向量是相等的."
```
但是默认赋值符`=`和三目条件运算符`a?b:c`都不可重载。
- 自定义运算符
除了标准的运算符之外，Swift还规定对只有`/ = - + * / < > ! & | ^ . ~`这些符号的运算符进行自定义。
新的运算符需要在全局域用`operator`关键字声明，声明为中置，前置或后置。
比如`operator prefix +++ {}`声明了新的前置运算符`+++`。
然后重载实现`+++`运算符：
``` swift
@prefix @assignment func +++ (inout vector: Vector2D) -> Vector2D {
    vector += vector
    return vector
}

var toBeDoubled = Vector2D(x: 1.0, y: 4.0)
let afterDoubling = +++toBeDoubled
// toBeDoubled 现在是 (2.0, 8.0)
// afterDoubling 现在也是 (2.0, 8.0)
```
还可以为自定义的运算符定义结合性和优先级。
结合性`associativity`后面可以接`left`（和左边操作数结合），`right`（和右边操作数结合），`none`（默认值，不与其他相同优先级的运算符写在一起）。
优先级`precedence`后面接数值表示优先级，默认为`100`。
下面一个例子。
``` swift
operator infix +- { associativity left precedence 140 }
func +- (left: Vector2D, right: Vector2D) -> Vector2D {
    return Vector2D(x: left.x + right.x, y: left.y - right.y)
}
let firstVector = Vector2D(x: 1.0, y: 2.0)
let secondVector = Vector2D(x: 3.0, y: 4.0)
let plusMinusVector = firstVector +- secondVector
// plusMinusVector 此时的值为 (4.0, -2.0)
```
自定义运算符优先级默认为100，其他标准运算符优先级从高到低：
- 160（无结合）：`<<`  `>>`
- 150（左结合）：`*`  `/`  `%`  `&*`  `&/`  `&%`  `&`（位与）
- 140（左结合）：`+`  `-`  `&+`  `&-`  `|`（位或）  `^`（位异或）
- 135（无结合）：`..<`  `...`
- 132（无结合）：`is`  `as`
- 130（无结合）：`<`  `<=`  `\>`  `\>=`  `==`  `!=`  `===`  `!==`  `~=`（模式匹配）
- 120（左结合）：`&&`（逻辑与）
- 110（左结合）：`||`（逻辑或）
- 100（右结合）：`? :` （三元条件）
- 90（右结合）：`=`  `*=`  `/=`  `%=`  `+=`  `-=`  `<<=`  `>>=`  `&=`  `|=`  `^=`  `&&=`  `||=`
