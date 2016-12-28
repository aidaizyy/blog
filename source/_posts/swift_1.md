title: "Swift基础入门(1)：常量变量，基本数据类型和基本运算符"
date: 2015-07-14 15:23:13
tags:
- swift
categories: swift 
toc: true
---

Swift是苹果公司于2014年推出的用于iOS，OS X和watchOS应用开发的新语言。
基于Swift 1.2。
本篇介绍Swift的基础知识：常量变量，基本数据类型和基本运算符。

<!--more-->
**Title: [Swift基础入门(1)：常量变量，基本数据类型和基本运算符](https://aidaizyy.github.io/swift_1)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2015-07-15](http://aidaizyy.github.io)**

## 概要

Swift结合了C和Objectiv-C的特点，基于Cocoa和Cocoa Touch框架。
本文主要讲述Swift的基本语法。

## 常量变量

### 命名

常量变量命名不能包括数学符号，箭头，保留的Unicode码位，连线和制表符，不能以数字开头。

### 声明

声明常量使用`let`关键字，声明变量使用`var`关键字。
一般可省略数据类型，通过赋予的第一个值来自动确定数据类型。
``` swift
let maxNumber = 20
var currentNumber = 0, item = 0.1
```
在swift中，语句结束不需要加分号。（添加分号也没有问题）
上面两句，声明了常量`maxNumber`，并赋值为20，这个值不能被改变。声明时可以不赋值，但之后只能赋值一次。
声明了变量`currentNumber`和`item`，并赋值为0和0.1，可赋值多次。
在一行中可声明多个常量或变量，用逗号隔开。
`maxNumber`和`currentNumber`第一次赋值了整数，被确定为整数类型`Int`；`item`第一次赋值了小数，被确定为浮点数类型`Double`（未指定数据类型时，小数一定会被确定为`Double`而不是`Float`）。

> 基本数据类型：
- Int
- Double
- Float
- Bool
- String
- Character

声明常量变量时，也可以指定数据类型，通过在常量变量名称后接冒号再接数据类型名称来实现。
``` swift
var currentNumer: Double = 5
println(currentNumber)	//输出currentNumber的值
```
上面两句输出结果为`5.0`，因为`currentNumber`指定为`Double`类型，即使给它赋值了整数5。

### 输出

`println`和`print`函数都是输出函数，区别在于前者在输出末尾加上了换行符。

输出常量变量：
``` swift
var currentNumer: Double = 5
println("The current number is \(currentNumber)")
```
上面两句输出结果为`The current number is 0.5`。
通过`\(常量变量)`将常量变量转换为字符串并在`println`语句中输出。

### 注释

和C语言类似，注释分为单行注释`//`和多行注释`/*  */`
``` swift
var single	//单行注释

/* 多行注释 */
```
不一样的地方在于，swift的`/* */`可以嵌套。

## 基本数据类型

### 整数

整数分为`Int8`，`UInt8`，`Int16`，`UInt16`，`Int32`，`UInt32`，`Int64`，`UInt64`，分别对应8，16，32，64位的有符号整数类型和无符号整数类型。
一般`Int`指`Int32`（32位电脑）或`Int64`（64位电脑）。
整数类型都有`min`和`max`两个方法。
``` swift
var tmp = Int.max
println(tmp)
```
结果为`9223372036854775807`（64位电脑）。

``` swift
let decimalInteger = 17		//十进制表示17
let binaryInteger = 0b1001	//二进制表示17
let octalInteger = 0o21		//八进制表示17
let hexadecimalInteger = 0x11	//十六进制表示17
```
二进制，八进制和十六进制分别加前缀`0b`，`0o`，`0x`表示。

### 浮点数

- Double：64位浮点数，至少15位数字
- Float：32位浮点数，最少6位数字

``` swift
let decimalDouble = 12.1875	//十进制表示12.1875
let exponentDouble = 1.21875e1	//十进制指数表示12.1875
let hexadecimalDouble = 0xC.3p0	//十六进制指数表示12.1875
```
浮点数字面量可以用十进制和十六进制表示，指数分别用`e`和`p`表示。

数值型字面量都可以加0或_，不影响数值，比如`000_1_000.000_000_1`等于`1000.0000001`。

### 布尔值

`Bool`有两个值`true`和`false`。

### 可选类型

>可选类型（_optionals)用来表示值可能丢失的情况：
- 有值且等于x
- 没有值

- 有无值判断
可以通过条件语句判断，`if optional != nil`，结果为`ture`即表示有值，否则表示无值。

- 强制解析
在名字后面加`!`强制获取可选类型的值，但必须在有值的情况下，否则会报错，`optional!`。

- 可选绑定
``` swift
let optionalValue: Int? = 123
if let actualValue = optionalValue {
} else {
}
```
`Int?`在数据类型后面加`?`表示包含该数据类型的可选类型，`optionalValue`表示包含`123`的可选类型，如果包含值，则赋值给`actualValue`，并返回`true`，否则返回`false`。

- 无值：nil
``` swift
var optionalInt: Int? = 123
optionalInt = nil

var optionalStr: String?
```
可选类型可以被赋值为nil，即表示无值，这表示一个确定的值。
如果可选类型声明时没有赋值，则自动赋值为nil。

- 隐式解析
声明时将数据类型后面的`?`改为`!`，表示一个隐式解析可选类型，即每次自动解析，使用时可直接用常量变量名称。

### 断言
可选类型无值可能会影响程序运行，在某些特定情况下，需要终止程序，我们使用断言。
断言类似于条件判断语句，不同点在于，结果为`false`时直接终止程序。
``` swift
let age = -3
assert(age >= 0, "age cannot be less than zero")
```
`assert`的第二个参数描述信息可以省略。

### 元组

元组（_tuples_）把多个数据类型组合成一个复合的数据类型。
``` swift
let httpStatus1 = (statusCode: 200, description: "OK")
println(httpStatus1.statusCode, httpStatus1.description)
//输出“200OK”

let httpStatus2 = (200, "OK")
println(httpStatus2.0, httpStatus2.1)
//输出“200OK”

let (statusCode, statusMessage) = httpStatus2
println(statusCode, statusMessage)
//输出“200OK”

let (statusCode, _) = httpStatus2
println(statusCode)
//输出“200”
```
元组用括号`(Int, String)`表示一个整数和一个字符串组合，可以给元组的单个元素命名，比如第1行的`statusCode`和`description`，调用时直接用`httpStatus1.statusCode`和`httpStatus2.description`表示；如果不命名，则用`.0`和`.1`表示。
也可以把元组内容分解，比如第9行，分别用`statusCode`和`statusMessage`存储元组`httpStatus2`对应的元素。分解过程中忽略的部分可用`_`表示，比如第13行，只使`statusCode`存储元组`httpStatus2`的第一个元素，忽略第二个元素。

### 类型别名

``` swift
typealias tmpType = Int
let tmpValue: tmpType = 4
```
通过`typealias`关键字，给现有的数据类型再起一个新的名字，可替代使用。
常量`tmpValue`的数据类型就是`Int`。

### 类型转换

- Int，Double，Float：
`Int16`与`Int8`不能直接相加，需要通过`Int16(Int8)`转换。
同样，`Double`与`Int`也不能相加，也需要通过`Double(Int)`转换，如果只需要整数部分，也可以通过`Int(Double)`转换。

- String，Int：
String->Int：`String.toInt()`函数可以把`String`转换成可选类型`Int?`，因为`String`中不一定能转换成`Int`，所以得到可选类型。
Int->String：`String(Int)`函数可以把`Int`转换成`String`。

## 基本运算符

### 普通运算符

大部分基本运算符和主流语言一致：
- +：加
- -：减
- *：乘
- /：除
- =：赋值
        - 不返回值，将`if a == b`误写成`if a = b`会出现编译错误。
        - 元组赋值，`let (x, y) = (1, 2)`，表示`x = 1`且`y = 2`。
- %：求余
        - 除了整数，也可以对浮点数求余，`8 % 2.5`等于`Double`值`0.5`。
- ++：自增
- --：自减
        - 除了整数，浮点数也可以自增和自减。
- -：负号
- +：正号
- +=, -=, *=, /=, %=：复合赋值
- ==：等于
- !=: 不等于
- \>, <, >=, <=：比较运算符
- ===, !===：是否引用同一个对象实例
- ? : ：三目运算符
- &&：与
- ||：或
- !：非
- ()：括号，确定运算先后顺序

### 空合运算符（Nil Coalescing Operator）

`a ?? b`：其中`a`必须是可选（_Optional_）类型，`b`的类型与a存储的值的类型一致。
如果a包含一个值，就返回`a`包含的值；否则返回默认值`b`，等同于`a != nil ? a! : b`。

### 区间运算符（Range Operator）

`a..<b`，闭区间运算符，表示`a`到`b`的区间，包含`a`，不包含`b`；
`a...b`，半开区间运算符，表示`a`到`b`的区间，包含`a`和`b`。

** 转载请注明原作者和出处。**
> 如果觉得这篇文章对您有帮助或启发，请随意打赏~
<p> <img src="http://7xivk7.com1.z0.glb.clouddn.com/paycode01.jpg" width = "250" align = "left" /> <img src="http://7xivk7.com1.z0.glb.clouddn.com/paycode02.jpg" width = "250" align = "left" /> </p>
