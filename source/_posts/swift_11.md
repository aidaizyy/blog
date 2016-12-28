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
**Last Modified: [2015-07-27](http://aidaizyy.github.io)**

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
协议类似于C++/Java语言中的接口，定义要实现的属性方法，但是不实现，由继承的枚举，结构体和类去实现。

## 语法
``` swift
protocol SomeProtocol {
    // 协议内容
}

struct SomeStructure: FirstProtocol, AnotherProtocol {
    // 结构体内容
}

class SomeClass: SomeSuperClass, FirstProtocol, AnotherProtocol {
    // 类的内容
}
```
结构体`SomeStructure`实现（用`:`表示）协议`someProtocol`和`AnotherProtocol`，多个协议隔开用`,`表示，需要实现协议的所有属性和方法。
类`SomeClass`实现父类`SomeSuperClass`和两个协议，需要把父类声明写到前面，协议并列写在后面。

- 协议类型
协议也可以作为一种基本类型，作为函数方法的参数类型、返回值类型，常量变量属性的类型，数组字典等集合的元素类型等等。
当协议作为集合的元素类型时，遍历集合得到的实例是协议类型，只能访问属于协议中定义的属性方法下标。

## 属性
协议中声明的属性，可以实现为实例属性或类型属性，存储型属性或计算型属性都可以。
协议中的只读属性，可以实现为只读属性或读写属性；但是协议中的读写属性，只能实现为读写属性。
``` swift
protocol SomeProtocol {
    var mustBeSettable : Int { get set }	//读写属性
    var doesNotNeedToBeSettable: Int { get }	//只读属性
}
```
协议中可以定义类型属性，只能实现为类型属性，属性前加上关键字`static`。
枚举和结构体实现后要在属性前加上关键字`static`，但是类实现后要在属性前加上关键字`class`。
``` swift
protocol AnotherProtocol {
    static var someTypeProperty: Int { get set }
}
```
下面一个实例。
``` swift
protocol FullyNamed {
    var fullName: String { get }
}

struct Person: FullyNamed{
    var fullName: String
}
let john = Person(fullName: "John Appleseed")
//john.fullName 为 "John Appleseed"
```

## 方法
协议中的方法不需要大括号和具体实现，支持变长参数，而不支持参数默认值。
``` swift
protocol RandomNumberGenerator {
    func random() -> Double
}
```
和类型属性一样，类型方法也是在协议中使用`static`，枚举和结构体中用`static`继承，而类中用`class`继承。
``` swift
protocol SomeProtocol {
    class func someTypeMethod()
}
```
- 变异方法
在`func`前加上关键字`mutating`的方法表示在该方法中可以修改实例及其属性的值，称为变异（_mutating_）方法。
枚举和结构体实现时，需要加上`mutating`关键字；而类实现时，不需要加上`mutating`关键字。
下面一个实例。
``` swift
protocol Togglable {
    mutating func toggle()
}

enum OnOffSwitch: Togglable {
    case Off, On
    mutating func toggle() {
        switch self {
        case Off:
            self = On
        case On:
            self = Off
        }
    }
}
var lightSwitch = OnOffSwitch.Off
lightSwitch.toggle()
//lightSwitch 现在的值为 .On
```
在`func toggle()`中，修改了`self`的值，属于变异方法。

## 构造器
协议中的构造器不需要大括号和具体实现。
类实现协议时，必须给构造器前面加上关键字`required`，表示它的子类必须继承的构造器。
当然，如果针对`final`类就不需要加`required`了，标有关键字`final`的类表示不能有子类。
``` swift
protocol SomeProtocol {
    init(someParameter: Int)
}

class SomeClass: SomeProtocol {
    required init(someParameter: Int) {
        //构造器实现
    }
}
```
还有一种情况，如果子类重写了父类的指定构造器，需要在构造器前加上`override`，但是如果又要同时实现协议，需要在构造器前加上`required`，那么`required`应该放在`override`前面。
``` swift
class SomeSubClass: SomeSuperClass, SomeProtocol {
    // "required" from SomeProtocol conformance; "override" from SomeSuperClass
    required override init() {
        // 构造器实现
    }
}
```
- 可失败构造器
协议中的可失败构造器可以实现成可失败构造器或非可失败构造器；而协议中的非可失败构造器只能实现成非可失败构造器或隐式解析类型的可失败构造器（`init!`）。

## 适配协议
扩展可以对已存在的枚举，结构体和类添加成员，比如属性，方法，下标，协议等。
下面的例子展示了通过扩展为已有类型适配协议。
``` swift
protocol TextRepresentable {
    func asText() -> String
}

extension Dice: TextRepresentable {
    func asText() -> String {
        return "A \(sides)-sided dice"
    }
}
```
`Dice`是一个已存在的类型，协议中的`asText`方法被添加实现在了`Dice`中。
如果`Dice`已经存在了`asText() -> String`方法，可以直接`extension Dice: TextRepresentable {}`，添加实现协议的声明。

## 协议继承
协议也可以和类一样，继承别的协议。
``` swift
protocol InheritingProtocol: SomeProtocol, AnotherProtocol {
    // 协议定义
}
```
- 类专属协议
如果在继承列表中加上`class`，且`class`必须第一个被声明，其他继承协议在后面并列声明，则表示这个协议只能由类实现，枚举和结构体不可以实现。
``` swift
protocol SomeClassOnlyProtocol: class, SomeInheritedProtocol {
    // class-only protocol definition goes here
}
```

## 协议合成
多个协议可以用protcol<someProtocol, AnotherProtocol>的格式临时组合成一个协议。
下面一个例子。
``` swift
protocol Named {
    var name: String { get }
}
protocol Aged {
    var age: Int { get }
}
struct Person: Named, Aged {
    var name: String
    var age: Int
}
func wishHappyBirthday(celebrator: protocol<Named, Aged>) {
    println("Happy birthday \(celebrator.name) - you're \(celebrator.age)!")
}
let birthdayPerson = Person(name: "Malcolm", age: 21)
wishHappyBirthday(birthdayPerson)
// 输出 "Happy birthday Malcolm - you're 21!
```

## 协议一致性
用`is`操作符检查协议是否实现了特定协议，返回`true`或`false`。
用`as?`操作符返回一个可选值，如果协议实现了特定协议，返回协议类型；否则返回`nil`。如果这个协议一定实现了特定协议，可以用`as!`强制返回非可选的特定类型。
下面一个例子。
``` swift
protocol HasArea {
    var area: Double { get }
}

class Circle: HasArea {
    let pi = 3.1415927
    var radius: Double
    var area: Double { return pi * radius * radius }
    init(radius: Double) { self.radius = radius }
}
class Country: HasArea {
    var area: Double
    init(area: Double) { self.area = area }
}
class Animal {
    var legs: Int
    init(legs: Int) { self.legs = legs }
}

let objects: [AnyObject] = [
    Circle(radius: 2.0),
    Country(area: 243_610),
    Animal(legs: 4)
]

for object in objects {
    if let objectWithArea = object as? HasArea {
        println("Area is \(objectWithArea.area)")
    } else {
        println("Something that doesn't have an area")
    }
}
// Area is 12.5663708
// Area is 243610.0
// Something that doesn't have an area
```
类`Circle`和类`Country`都实现了`HasArea`协议，而类`Animal`没有实现`HasArea`协议。
这三个类的实例用同一个数组装上，然后遍历，利用协议一致性检查。在这个数组中实例的值的类型没有变，但是这里显式为`HasArea`类型，所以只能访问`area`属性。

## 可选协议
可选类型可以含有可选成员，可以选择是否实现这些可选成员，用关键字`optional`来表示这些可选成员。可选协议在调用时可以使用可选链。
协议前的`@objc`表示协议是可选的，也表示暴露给`Objective-C`的代码，只对类有效。所以可选协议只能由类实现。
下面一个例子。
``` swift
@objc protocol CounterDataSource {
    optional func incrementForCount(count: Int) -> Int
    optional var fixedIncrement: Int { get }
}

@objc class Counter {
    var count = 0
    var dataSource: CounterDataSource?
    func increment() {
        if let amount = dataSource?.incrementForCount?(count) {
            count += amount
        } else if let amount = dataSource?.fixedIncrement? {
            count += amount
        }
    }
}

//为协议实现可选属性
@objc class ThreeSource: CounterDataSource {
    let fixedIncrement = 3
}
var counter = Counter()
counter.dataSource = ThreeSource()
for _ in 1...4 {
    counter.increment()
    print(counter.count)
}
// 3
// 6
// 9
// 12

//为协议实现可选方法
class TowardsZeroSource: CounterDataSource {
func incrementForCount(count: Int) -> Int {
        if count == 0 {
            return 0
        } else if count < 0 {
            return 1
        } else {
            return -1
        }
    }
}
counter.count = -4
counter.dataSource = TowardsZeroSource()
for _ in 1...5 {
    counter.increment()
    print(counter.count)
}
// -3
// -2
// -1
// 0
// 0
```

## 协议扩展
扩展协议可以为每个实现该协议的地方（_遵循者_）添加属性或方法的实现，该协议的遵循者不用任何修改，可以得到添加的属性方法。
这种方式可以为协议中的属性和方法提供默认的实现，遵循者中再次实现可以覆盖默认的实现。
扩展协议时可以限定条件，只有满足条件的遵循者能够得到协议扩展的属性和方法。

** 转载请注明原作者和出处。**
> 如果觉得这篇文章对您有帮助或启发，请随意打赏~
<p> <img src="http://7xivk7.com1.z0.glb.clouddn.com/paycode01.jpg" width = "250" align = "left" /> <img src="http://7xivk7.com1.z0.glb.clouddn.com/paycode02.jpg" width = "250" align = "left" /> </p>
