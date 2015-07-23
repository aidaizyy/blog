title: "Swift基础入门(3)：数组，集合和字典"
date: 2015-07-15 17:04:30
tags:
- swift
categories: swift
toc: true
---

本篇介绍Swift的基础知识：数组（Array），集合（Set）和字符（Dictionary）。

<!--more-->
**Title: [Swift基础入门(3)：数组，集合和字典](https://aidaizyy.github.io/swift_3)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2015-07-16](http://aidaizyy.github.io)**

# 数组

## 构造
``` swift
var shoppinglist = ["Eggs", "Milk"]	//初始化为字符串数组，没有指定数据类型，通过添加数据自动判断为String数组

var someInts1 = [2, 3]
var someInts2: [Int] = [2, 3]		//等价于上一句，初始化为整数数组，指定了数据类型Int，只能添加Int数据，并添加了元素2，3
var someInts3: Array<Int> = [2, 3]	//等价于上一句

var someInts4 = Array<Int>()		//初始化为整数数组，指定了数据类型Int，只能添加Int数据，没有添加元素
var someInts5 = [Int]()			//等价于上一句

var someDoubles = []			//初始化为空数组，没有指定数据类型，通过添加数据自动判断
someDoubles.append(2.3)			//通过添加数据自动判断为Double数组

var threeDoubles = [Double](count: 3, repeatedValue: 0.0)
//(count: , repeatedValue: )形式，指定了重复的值和重复的次数，构造数组{0.0, 0.0, 0.0}
```
数组的元素只能有一种数据类型。

## 数量
- Array.count：属性`count`表示数组`Array`的元素个数。
- Array.isEmpty：属性`isEmpty`表示数组`Array`的元素是否为0个，结果为`true`或`false`。

## 访问
- Array[i]：通过下标`[i]`访问数组`Array`的第`i`位，可修改。

## 遍历
``` swift
var shoppinglist = ["Eggs", "Milk"]
for item in shoppingList {
	println(item)
}
//Eggs
//Milk
```

## 添加
- Array.append(Item)：将元素`Item`添加到数组`Array`的尾部。
- Array.imsert(Item, atIndex: i)：将元素`Item`添加到数组`Array`的第`i`位。
- Array += [Item1, Item2]：将元素`Item1`和`Item2`添加到数组`Array`的尾部。

## 删除
- Array.removeAtIndex(i)：删除数组`Array`的第`i`位。
- Array.removeLast()：删除数组`Array`的最后一位。

## 替换
- Array[m...n] = [Item1, Item2]：用元素`Item1`和`Item2`替换数组`Array`的第`m`位到第`n`位。这种方法不能用于添加新元素。

#  集合
集合中的元素没有确定顺序，且每个元素只出现一次。

## 构造
``` swift
var shoppinglist: Set = ["Eggs", "Milk"]	//初始化为字符串集合，没有指定数据类型，通过添加数据自动判断为String集合

var someInts1: Set = [2, 3]			
var someInts2: Set<Int> = [2, 3]		//等价于上一句，初始化为整数集合，指定了数据类型Int，只能添加Int数据，并添加了元素2，3

var someInts3 = Set<Int>()			//初始化为整数集合，指定了数据类型Int，只能添加Int数据，没有添加元素

var someDoubles: Set = []			//初始化为空集合，没有指定数据类型，通过添加数据自动判断
someDoubles.insert(2.3)				//通过添加数据自动判断为Double集合
```
集合的元素只能有一种数据类型。

## 数量
- Set.count：属性`count`表示集合`Set`的元素个数。
- Set.isEmpty：属性`isEmpty`表示集合`Set`的元素是否为0个，结果为`true`或`false`。

## 遍历
``` swift
var shoppinglist1: Set = ["Milk", "Eggs"]
for item in shoppingList {
	println(item)
}
//Milk
//Eggs

var shoppinglist2: Set = ["Milk", "Eggs"]
for item in sorted(shoppingList) {
	println(item)
}
//Eggs
//Milk
```
因为`Set`中没有确定顺序，可以通过`sorted(Set)`函数返回一个排序的集合。

## 添加
- Set.imsert(Item)：将元素`Item`添加到集合`Set`中。

## 删除
- Set.remove(Item)：删除集合`Set`中的元素`Item`，成功则返回`Item`，如果集合中不包含`Item`则返回`nil`。
- Set.removeAll()：删除集合`Set`中的所有元素

## 包含
- Set.contains(Item)：检查集合`Set`是否包含元素`Item`，返回`true`或`false`。

## 比较
``` swift
let s1: Set = [1, 2]
let s2: Set = [3, 4, 5, 1, 2]
let cityAnimals: Set = [6, 7]
s1.isSubsetOf(s2)
// true
s2.isSuperSetOf(s1)
// true
s2.isDisjointWith(s3)
// true
```
- ==：判断两个集合是否相等
- Set1.isSubsetOf(Set2)：判断`Set1`是否是`Set2`的子集
- Set1.isSupersetOf(Set2)：判断`Set1`是否是`Set2`的父集
- Set1.isStrictSubsetOf(Set2)，Set1.isStrictSupersetOf(Set2)：和上面方法相似，不过两个集合不能相等。
- Set1.isDisjoinWith(Set2)：判断`Set1`和`Set2`是否完成没有一个相同元素
上述方法都返回`true`或`false`。

## 操作
``` swift
let oddDigits: Set = [1, 3, 5, 7, 9]
let evenDigits: Set = [0, 2, 4, 6, 8]
let singleDigitPrimeNumbers: Set = [2, 3, 5, 7]
sorted(oddDigits.union(evenDigits))
// [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
sorted(oddDigits.intersect(evenDigits))
// []
sorted(oddDigits.subtract(singleDigitPrimeNumbers))
// [1, 9]
sorted(oddDigits.exclusiveOr(singleDigitPrimeNumbers))
// [1, 2, 9]
```
- Set1.intersects(Set2)：返回`Set1`和`Set2`的交集，即两个集合中都有的元素
- Set1.union(Set2)：返回`Set1`和`Set2`的并集，即两个集合中的所有元素
- Set1.subtract(Set2)：返回`Set1`和`Set2`的差集，即`Set1`中有的且`Set2`中没有的元素
- Set1.exclusiverOr(Set2)：返回并集减去并集的集合，即`Set1`中独有的和`Set2`中独有的元素，也就是所有元素减去两个集合中都有的元素。

## 哈希值
Swift中的所有基本类型默认都是可哈希的，通过`a.hashValue`求得哈希值。哈希值相等可以判断对象相同，如`a == b`即`a.hashValue == b.hashValue`。

# 字典
字典中每个值（_Value_）都关联唯一的建（_key_）。

## 构造
在构造过程中，键值对默认用`[key 1: value 1, key 2: value 2, key 3: value 3]`的形式。
``` swift
var airports1 = ["TYO": "Tokyo", "DUB": "Dublin"]				//初始化为[String: String]字典，没有指定数据类型，通过添加数据自动判断
var airports2: [String: String] = ["TYO": "Tokyo", "DUB": "Dublin"]		//等价于上一句
var airports3: Dictionary<Stringr, String> = ["TYO": "Tokyo", "DUB": "Dublin"]	//等价于上一句

var airports4 = Dictionary<String, Sting>()	//初始化为[Sting: String]空字典，指定了数据类型[String: String]，只能添加[String:String]数据，没有添加元素
var airports5 = [String: String]()		//等价于上一句

var airports6 = [:]	//初始化为空字典，没有指定数据类型，通过添加数据自动判断
airports6[2] = 3	//通过添加数据自动判断为[Int: Int]字典
```

## 数量
- Dictionary.count：属性`count`表示字典`Dictionary`的元素个数。
- Dictionary.isEmpty：属性`isEmpty`表示字典`Dictionary`的元素是否为0个，结果为`true`或`false`。

## 访问
- Dictionary[key]：通过下标`[key]`访问字典`Dictionary`的键`key`对应的值，可修改。

## 遍历
``` swift
for (airportCode, airportName) in airports {
    println("\(airportCode): \(airportName)")
}
// TYO: Tokyo
// DUB: Dublin

for airportCode in airports.keys {
    println("Airport code: \(airportCode)")
}
// Airport code: TYO
// Airport code: DUB

for airportName in airports.values {
    println("Airport name: \(airportName)")
}
// Airport name: Tokyo
// Airport name: Dublin
```
`for-in`可便利字典，可便利键值对`(key, value)`，也可以通过属性`keys`或`values`只便利键值其中一项。
字典的属性`keys`和`values`返回数组。
``` swift
let airportCodes = Array(airports.keys)
// airportCodes is ["TYO", "DUB"]

let airportNames = Array(airports.values)
// airportNames is ["Tokyo", "Dublin"]
```
## 添加
- Dictionary[key] = value：更新字典`Dictionary`中键`key`对应的值，如果不存在，则将键值对<key, valye>添加到字典`Dictionary`中。
- Dicitonary.updateValue(value, forkey: key)：更新字典`Dictionary`中键`key`对应的值，如果不存在，则将键值对<key, value>添加到字典`Dictionary`。
注意：该方法返回**原值**，即执行`updateValue`方法之前键`key`对应的值，如果不存在，则返回`nil`。

## 删除
- Dictionary[key] = nil：删除字典`Dictionary`中键`key`对应的值。
- Dictionary.removeValueForKey(key)：删除字典`Dictionary`中键`key`对应的值，返回删除的值，如果不存在，则返回`nil`。

添加操作和删除操作返回值有可能为`nil`，都是可选类型，使用时需要进行判断是否有值。
