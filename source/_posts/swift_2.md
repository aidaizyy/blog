title: "Swift基础入门(2)：字符串和字符"
date: 2015-07-15 14:09:04
tags:
- swift
categories: swift
toc: true
---

本篇介绍Swift的基础知识：字符串和字符。

<!--more-->
**Title: [Swift基础入门(2)：字符串和字符](https://aidaizyy.github.io/swift_2)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2015-07-15](http://aidaizyy.github.io)**

## 字符串和字符

### 空字符串
``` swift
var str1 = ""
var str2 = String()

if str1.isEmpty {
	//空字符串
}
```
两条语句等价，都表示空字符串。
`String`的`isEmpty`属性表示`String`是否为空，结果为`Bool`值。

### 值传递
在函数/方法中传递的是字符串的值，不会改变字符串本身。

### 遍历
``` swift
for character in "Dog!" {
    println(character)
}
// D
// o
// g
// !
```
`for-in`：`for characte in "Hello World!"`将会遍历字符串`"Hello World!"`的每个字符，并用`character: Character`来表示。

### 长度
`count(String)`函数，得到字符串的字符数量。

### 连接
- +, +=：连接字符串
- String.append(Character)：将字符连接到字符串尾部。

### 比较
- ==：字符串相等
- String1.hasPrefix(String2)：是否有特定前缀。如果`String1`包含前缀`String2`返回`true`，否则返回`false`。
- String1.hasSuffix(String2)：是否有特定后缀。如果`String1`包含后缀`String2`返回`true`，否则返回`false`。

### 大小写
``` swift
let normal = "Could you help me, please?"
let shouty = normal.uppercaseString
// shouty 值为 "COULD YOU HELP ME, PLEASE?"
let whispered = normal.lowercaseString
// whispered 值为 "could you help me, please?"
```
String.uppercaseString属性表示字符串的大写，String.lowercaseString属性表示字符串的小写。

## Unicode

Unicode字符用`\u{n}`表示，其中`n`为任意的一到八位十六进制数。

>String：
属性：
String.isEmpty
String.uppercaseString
String.lowercaseString
String.utf8
String.utf16
String.unicodeScalars
方法：
String.append()
String.hasPrefix()
String.hasSuffix()
count(String)
