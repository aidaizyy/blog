title: "Swift基础入门(7)：属性，方法和下标"
date: 2015-07-20 10:10:20
tags:
- swift
categories: 
toc: true
---

本篇介绍Swift的基础知识：枚举，结构体和类的属性，方法和下标。

<!--more-->
**Title: [Swift基础入门(7)：属性，方法和下标](https://aidaizyy.github.io/swift_7)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2015-07-24](http://aidaizyy.github.io)**

# 属性
属性分为存储属性（只能用于类和结构体）和计算属性（可用于类，结构体和枚举）。

## 存储属性
存储属性可用`var`或`let`修饰。
结构体用`let`修饰，属性不可以更改，因为结构体是值类型。
类用`let`修饰，属性可以更改，因为类是引用类型。
结构体可以在构造时逐一初始化属性，而类不可以，参见[类和结构体](http://aidaizyy.github.io/swift_6/#类和结构体)。
- 延迟属性
延迟属性用`lazy`标示，且必须使用`var`关键字，只有在第一次被调用时才会计算其初始值，在构造时不会计算初始值。
``` swift
class DataImporter {
    /*
    DataImporter 是一个将外部文件中的数据导入的类。
    这个类的初始化会消耗不少时间。
    */
    var fileName = "data.txt"
    // 这是提供数据导入功能
}

class DataManager {
    lazy var importer = DataImporter()
    var data = [String]()
    // 这是提供数据管理功能
}

let manager = DataManager()
manager.data.append("Some data")
manager.data.append("Some more data")
// DataImporter 实例的 importer 属性还没有被创建

println(manager.importer.fileName)
// DataImporter 实例的 importer 属性现在被创建了
// 输出 "data.txt”
```
以上并未给出全部代码，类`DataManager`中声明了延迟属性`importer`。
类`DataImporter`实现数据导入功能，会消耗不少时间。
初始化类`DataManager`时，延迟属性并不会创建。
只有在`println(manager.importer.fileName)`时，属性`importer`第一次被调用时，才会创建延迟属性`importer`，完成数据导入功能。

## 计算属性
计算属性不直接存储值，提供getter获取值和可选的setter来间接设置其他属性的值，必须用`var`修饰。
``` swift
struct Point {
    var x = 0.0, y = 0.0
}
struct Size {
    var width = 0.0, height = 0.0
}
struct Rect {
    var origin = Point()
    var size = Size()
    var center: Point {
    get {
        let centerX = origin.x + (size.width / 2)
        let centerY = origin.y + (size.height / 2)
        return Point(x: centerX, y: centerY)
    }
    set(newCenter) {
        origin.x = newCenter.x - (size.width / 2)
        origin.y = newCenter.y - (size.height / 2)
    }
    }
}
var square = Rect(origin: Point(x: 0.0, y: 0.0),
    size: Size(width: 10.0, height: 10.0))
let initialSquareCenter = square.center
square.center = Point(x: 15.0, y: 15.0)
println("square.origin is now at (\(square.origin.x), \(square.origin.y))")
// 输出 "square.origin is now at (10.0, 10.0)”
```
在结构体`Rect`中，属性`center`是计算属性，分别设置了`get`方法和`set`方法。
`set`方法可以不指定新值的参数名称，比如`newCenter`，在方法中直接使用默认名称`newValue`，上面可写作：
``` swift
set {
        origin.x = newValue.x - (size.width / 2)
        origin.y = newValue.y - (size.height / 2)
    }
```
- 只读计算属性
不设置setter，只设置getter的计算属性称为只读计算属性。
``` swift
struct Cuboid {
    var width = 0.0, height = 0.0, depth = 0.0
    var volume: Double {
    return width * height * depth
    }
}
let fourByFiveByTwo = Cuboid(width: 4.0, height: 5.0, depth: 2.0)
println("the volume of fourByFiveByTwo is \(fourByFiveByTwo.volume)")
// 输出 "the volume of fourByFiveByTwo is 40.0"
```
只读计算属性可省略`get`关键字。

## 属性观察器
属性观察器可以监控和响应属性值的变化。
- willSet 在设置新值之前调用
- didSet 在设置新值之后调用
``` swift
class StepCounter {
    var totalSteps: Int = 0 {
    willSet(newTotalSteps) {
        println("About to set totalSteps to \(newTotalSteps)")
    }
    didSet(oldTotalSteps) {
        if totalSteps > oldTotalSteps  {
            println("Added \(totalSteps - oldTotalSteps) steps")
        }
    }
    }
}
let stepCounter = StepCounter()
stepCounter.totalSteps = 200
// About to set totalSteps to 200
// Added 200 steps
stepCounter.totalSteps = 360
// About to set totalSteps to 360
// Added 160 steps
stepCounter.totalSteps = 896
// About to set totalSteps to 896
// Added 536 steps
```
`willSet`如果不指定新值参数名，可用`newValue`替代。
`didSet`如果不指定旧值参数名，可用`oldValue`替代。
通过重写的方式可以为继承来的存储属性和计算属性添加属性观察器。

计算属性和属性观察器也可以用于全局变量和局部变量。
全局的常量变量都是延迟计算的，不需要标记`lazy`。而局部的常量变量不会延迟计算。

## 类型属性
类型属性指所有类型实例公用的属性，类似于其他语言中的静态属性（_static_）。
值类型可以定义存储型和计算型的类型属性。
因为类是引用类型，所以只能定义计算型的类型属性。
存储型的类型属性必须指定默认值。
类型属性在属性前加上`static`关键字。
``` swift
struct SomeStructure {
    static var storedTypeProperty = "Some value."
    static var computedTypeProperty: Int {
    // 这里返回一个 Int 值
    }
}
enum SomeEnumeration {
    static var storedTypeProperty = "Some value."
    static var computedTypeProperty: Int {
    // 这里返回一个 Int 值
    }
}
class SomeClass {
    class var computedTypeProperty: Int {
    // 这里返回一个 Int 值
    }
}

println(SomeClass.computedTypeProperty)
// 输出 "42"

println(SomeStructure.storedTypeProperty)
// 输出 "Some value."
SomeStructure.storedTypeProperty = "Another value."
println(SomeStructure.storedTypeProperty)
// 输出 "Another value.”
```
这里计算型类型属性都是只读型，也可以定义为可读可写。

# 方法

## 实例方法
方法是定义在类，结构体和枚举中的方法，和函数类似。
类，结构体和枚举创建实例后，其中的方法被称为实例方法，只属于当前实例。
``` swift
class Counter {
  var count = 0
  func increment() {
    count++
  }
  func incrementBy(amount: Int) {
    count += amount
  }
  func reset() {
    count = 0
  }
}

let counter = Counter()
// 初始计数值是0
counter.increment()
// 计数值现在是1
counter.incrementBy(5)
// 计数值现在是6
counter.reset()
// 计数值现在是0
```
### 外部参数名
方法默认第一个参数没有外部参数名，第二个及以后参数默认有外部参数名，和参数名一致，相当于默认在参数前加上了`#`。
``` swift
class Counter {
  var count: Int = 0
  func incrementBy(amount: Int, numberOfTimes: Int) {
    count += amount * numberOfTimes
  }
}

let counter = Counter()
counter.incrementBy(5, numberOfTimes: 3)
// counter value is now 15
```
也可以在第一个参数名添加外部参数名，也可以用_放在第二个及以后的参数名前取消默认的外部参数名。
``` swift
class Counter {
    var count: Int = 0
    func incrementBy(#amount: Int, _ numberOfTimes: Int) {
        count += amount * numberOfTimes
    }
}

let counter = Counter()
counter.incrementBy(amount: 5, 3)
println(counter.count)
// counter value is now 15
```
关于外部参数名，参见[函数的外部参数名](http://aidaizyy.github.io/swift_5/#外部参数名)。

### self属性
在每个实例中，都有一个隐藏属性`self`，指代实例变身，以便方法调用实例本身。
``` swift
func increment() {
  self.count++
}
```

### 变异方法
结构体和枚举是值类型。值类型的属性不可以在实例方法中被修改。
变异方法可以完成对属性的修改，在`func`前加上`mutating`关键字。
``` swift
struct Point {
  var x = 0.0, y = 0.0
  mutating func moveByX(deltaX: Double, y deltaY: Double) {
    x += deltaX
    y += deltaY
  }
}
var somePoint = Point(x: 1.0, y: 1.0)
somePoint.moveByX(2.0, y: 3.0)
println("The point is now at (\(somePoint.x), \(somePoint.y))")
// 输出 "The point is now at (3.0, 4.0)"
```
变异方法也可以给`self`赋值，即新建一个实例替代旧的实例。
``` swift
struct Point {
  var x = 0.0, y = 0.0
  mutating func moveByX(deltaX: Double, y deltaY: Double) {
    self = Point(x: x + deltaX, y: y + deltaY)
  }
}
```
``` swift
enum TriStateSwitch {
  case Off, Low, High
  mutating func next() {
    switch self {
    case Off:
      self = Low
    case Low:
      self = High
    case High:
      self = Off
    }
  }
}
var ovenLight = TriStateSwitch.Low
ovenLight.next()
// ovenLight 现在等于 .High
ovenLight.next()
// ovenLight 现在等于 .Off
```
类是引用类型，实例方法可以直接修改属性。

## 类型方法
类型方法和类型属性类似，都是指所有类型实例公共的方法，类似于其他语言中的静态方法（_static_）。
类型方法在类型前加上`class`关键字。
类型方法能够直接通过静态属性的名称访问静态属性。
``` swift
class SomeClass {
  class func someTypeMethod() {
    // type method implementation goes here
  }
}
SomeClass.someTypeMethod()
```

# 下标
下标（_subscripts_）可以定义在类，结构体和枚举中，是访问对象，集合和序列的快捷方式。比如[数组的访问](http://aidaizyy.github.io/swift_3/#访问)：Array[index]，[字典的访问](http://aidaizyy.github.io/swift_3/#访问-1)：Dictionary[key]。
下标的定义类似于实例方法和计算性属性的混合。
使用`subscript`关键字，定义了传入参数数量和类型和返回类型，定义了getter和setter。
``` swift
subscript(index: Int) -> Int {
    get {
      // 返回与入参匹配的Int类型的值
    }

    set(newValue) {
      // 执行赋值操作
    }
}
```
getter和setter的定义和计算型属性一样。
setter中可以使用`newValue`默认值，可以省略setter定义成只读类型。
``` swift
struct TimesTable {
    let multiplier: Int
    subscript(index: Int) -> Int {
      return multiplier * index
    }
}
let threeTimesTable = TimesTable(multiplier: 3)
println("3的6倍是\(threeTimesTable[6])")
// 输出 "3的6倍是18"
```
下标允许任意数量的传入参数，任意类型的传入参数和任意类型的返回值。
可以使用[变量参数](http://aidaizyy.github.io/swift_5/#变量参数)和[可变参数](http://aidaizyy.github.io/swift_5/#可变参数)，但是不能使用[输入输出参数（inout）](http://aidaizyy.github.io/swift_5/#输入输出参数)和[默认参数值](http://aidaizyy.github.io/swift_5/#默认参数值)。
``` swift
struct Matrix {
    let rows: Int, columns: Int
    var grid: [Double]
    init(rows: Int, columns: Int) {
      self.rows = rows
      self.columns = columns
      grid = Array(count: rows * columns, repeatedValue: 0.0)
    }
    func indexIsValidForRow(row: Int, column: Int) -> Bool {
        return row >= 0 && row < rows && column >= 0 && column < columns
    }
    subscript(row: Int, column: Int) -> Double {
        get {
            assert(indexIsValidForRow(row, column: column), "Index out of range")
            return grid[(row * columns) + column]
        }
        set {
            assert(indexIsValidForRow(row, column: column), "Index out of range")
            grid[(row * columns) + column] = newValue
        }
    }
}

var matrix = Matrix(rows: 2, columns: 2)
println(matrix[0, 1])
//0.0
matrix[0, 1] = 1.5
println(matrix[0, 1])
//1.5
```
