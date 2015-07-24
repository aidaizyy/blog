title: "Swift基础入门(8)：继承，构造，析构和嵌套类型"
date: 2015-07-21 17:17:02
tags:
- swift
categories: swift
toc: true
---

本篇介绍Swift的基础知识：类的继承；枚举，结构体和类的构造过程，析构过程和嵌套类型。

<!--more-->
**Title: [Swift基础入门(8)：继承，构造，析构和嵌套类型](https://aidaizyy.github.io/swift_8)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2015-07-24](http://aidaizyy.github.io)**

# 继承

## 基本语法
子类（_subclass_）继承（_inherit_）继承超类/父类（_superclass_）的属性，方法，下标和其他特性。
声明子类时，将超类名写在子类名的后面，用冒号分割：
``` swift
class Vehicle {
    var currentSpeed = 0.0
    var description: String {
        return "traveling at \(currentSpeed) miles per hour"
    }
    func makeNoise() {
        // 什么也不做-因为车辆不一定会有噪音
    }
}

class Bicycle: Vehicle {
    var hasBasket = false
}

class Tandem: Bicycle {
    var currentNumberOfPassengers = 0
}

let tandem = Tandem()
tandem.hasBasket = true
tandem.currentNumberOfPassengers = 2
tandem.currentSpeed = 22.0
println("Tandem: \(tandem.description)")
// Tandem: traveling at 22.0 miles per hour
```

## 重写
重写（_overriding_）指子类把父类的实例方法，类方法，实例属性和下表脚本等提供自己定制的实现。
在重写定义的前面加上关键字`override`。
使用`super`前缀可以访问超类的属性，方法和下表脚本。

### 重写方法
``` swift
class Train: Vehicle {
    override func makeNoise() {
        println("Choo Choo")
    }
}

class Car: Vehicle {
    var gear = 1
    override var description: String {
        return super.description + " in gear \(gear)"
    }
}

let train = Train()
train.makeNoise()
// prints "Choo Choo"
```

### 重写属性
- 超类的只读属性在子类中可以重写为读写属性，但是读写属性不能重写为只读属性。
- 超类的重写属性在子类中必须完整实现setter和getter，可以用`super.someProperty`返回超类的getter。
``` swift
class Car: Vehicle {
    var gear = 1
    override var description: String {
        return super.description + " in gear \(gear)"
    }
}

let car = Car()
car.currentSpeed = 25.0
car.gear = 3
println("Car: \(car.description)")
// Car: traveling at 25.0 miles per hour in gear 3
```

### 重写属性观察器
setter和属性观察器不能同时存在，setter中可以观察到值的变化。
``` swift
class AutomaticCar: Car {
    override var currentSpeed: Double {
        didSet {
            gear = Int(currentSpeed / 10.0) + 1
        }
    }
}

let automatic = AutomaticCar()
automatic.currentSpeed = 35.0
println("AutomaticCar: \(automatic.description)")
// AutomaticCar: traveling at 35.0 miles per hour in gear 4
```

## 防止重写
属性，方法和下标前面加上`final`关键字可以防止它们被重写。
`final var`，`final func`，`final class func`，`final subscript`。

# 构造过程
构造过程（_Inititalization_）为实例的每个属性设置初始值和为其执行必要的准备和初始化任务。

## 构造器
### 属性默认值
属性声明时，可以为其设置默认值。
``` swift
struct Fahrenheit {
    var temperature = 32.0
}

var f = Fahrenheit()
println("The default temperature is \(f.temperature)° Fahrenheit")
// 输出 "The default temperature is 32.0° Fahrenheit”
```
构造器，也可以为属性赋初始值，关键字`init`。
``` swift
struct Fahrenheit {
    var temperature: Double
    init() {
        temperature = 32.0
    }
}

var f = Fahrenheit()
println("The default temperature is \(f.temperature)° Fahrenheit")
// 输出 "The default temperature is 32.0° Fahrenheit”
```

### 构造器参数
构造器可以传入参数。
传入参数默认具有和内部参数名一致的外部参数名，相当于默认在参数名前加上了`#`。
用`_`替代外部参数名，可以取消默认的外部参数名。
``` swift
struct Color {
    let red, green, blue: Double
    init(red: Double, g green: Double, _ blue: Double) {
        self.red   = red
        self.green = green
        self.blue  = blue
    }
    init(white: Double) {
        red   = white
        green = white
        blue  = white
    }
}

let magenta = Color(red: 1.0, g: 0.0, 1.0)
let halfGray = Color(white: 0.5)
```

### 可选类型属性
如果属性为可选类型，构造器自动初始化为`nil`。

### 常量属性
构造器中可以修改常量`let`属性的值，在构造过程中结束后，常量的值不能被修改。

### 默认构造器
所有属性已提供默认值且没有定义构造器的结构体或基类，具有一个默认的构造器，把默认值赋值给属性作为初始值。

前面讲过，结构体的逐一成员构造器，算是一个默认的构造器。
``` swift
struct Size {
    var width = 0.0, height = 0.0
}
let twoByTwo = Size(width: 2.0, height: 2.0)
```
这里`Size`获得了一个逐一成员构造器`init(width: height: )`。

## 值类型的构造器代理
构造器可以通过调用其他构造器来完成构造过程，称为构造器代理。
值类型比较简单，只能调用本身提供的其他构造器，而类可以继承构造器。
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
    init() {}
    init(origin: Point, size: Size) {
        self.origin = origin
        self.size = size
    }
    init(center: Point, size: Size) {
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
        self.init(origin: Point(x: originX, y: originY), size: size)
    }
}
```
结构体`Rect`中实现了三个构造器。
第一个构造器功能和默认构造器类似，把默认值赋值给属性。
第二个构造器功能和逐一成员构造器类似，逐一把值赋值给属性。
第三个构造器调用了第二个构造器，完成了部分构造过程。

## 类的构造器代理

### 指定构造器和便利构造器
类类型的构造器要确保所有存储型属性获得初始值，包括继承来的属性，分为指定构造器和便利构造器。
- 指定构造器
主要的类构造器，根据父类链依次往上调用父类的构造器，每个类都必须拥有至少一个指定构造器。
写法和值类型的构造器一样：
``` swift
init(parameters) {
    statements
}
```
- 便利构造器
次要的类构造器，调用同一个类中的指定构造器，也可以创建一个特殊用途或特定输入的实例，只在必要时提供便利构造器。
写法和值类型的构造器也基本一样，在`init`前加上`convenience`关键字：
``` swift
convenience init(parameters) {
    statements
}
```

+ 指定构造器必须调用其直接父类的指定构造器。
+ 便利构造器必须调用同一类中定义的其他构器。
+ 便利构造器必须最终以调用一个指定构造器结束。

也就是说：
- 指定构造器必须总是向上代理
- 便利构造器必须总是横向代理

### 构造器继承和重写
重写（_Override_）指定构造器，在子类中重写实现并调用父类构造器。
重写便利构造器，必须通过调用同一类提供的其他指定构造器来实现。

子类不会默认继承父类的构造器。
如果特定条件满足，父类构造器也会被自动继承：
- 子类的任意新属性都有默认值，且没有定义任何指定构造器，它将自动继承所有父类的指定构造器。
- 子类提供了所有父类指定构造器的实现，它将自动继承所有父类的便利构造器。

### 构造过程
构造过程分为两个阶段。
第一个阶段：
沿着构造器链先初始化子类的属性，再代理给父类构造器，初始化父类的属性。
当到达构造器链最顶部时，所有的存储型属性都已经赋值。
这个阶段不能调用任何实例方法，不能读取任何实例属性的值，不能引用`self`的值。

第二个阶段：
沿着构造器链沿相反方向，从顶部向下，进一步定制实例，可以为任意属性赋新值。
这个阶段可以调用实例方法，修改实例属性，并访问`self`。

### 实例
``` swift
class Food {
    var name: String
    init(name: String) {
        self.name = name
    }
    convenience init() {
        self.init(name: "[Unnamed]")
    }
}

let namedMeat = Food(name: "Bacon")
// namedMeat 的名字是 "Bacon”

let mysteryMeat = Food()
// mysteryMeat 的名字是 [Unnamed]
```
类`Food`提供了一个指定构造器`init(name: String)`和一个便利构造器`init()`。
第11行：指定构造器，初始化属性`name`，因为`Food`没有父类，所以结束构造过程。
第14行：便利构造器，调用了同一个类的指定构造器并给参数`name`传入值`[Unnamed]`。
``` swift
class RecipeIngredient: Food {
    var quantity: Int
    init(name: String, quantity: Int) {
        self.quantity = quantity
        super.init(name: name)
    }
    override convenience init(name: String) {
        self.init(name: name, quantity: 1)
    }
}

let sixEggs = RecipeIngredient(name: "Eggs", quantity: 6)
let oneBacon = RecipeIngredient(name: "Bacon")
let oneMysteryItem = RecipeIngredient()
```
类`RecipeIngredient`继承类`Food`，提供了一个指定构造器`init(name: String, quantity: Int)`和一个便利构造器`init(name: String)。
第12行：指定构造器，先初始化子类的属性`quantity`，再代理给父类`Food`的`init(name: String)`。
第13行：便利构造器，调用了同一个类的指定构造器并给参数`name`和`quantity`传入了值。
因为`init(name: String)`和父类的`init(name: String)`使用了相同的参数，所以在前面使用`override`标识。
第14行：父类的`init()`被子类继承了，但是它其中调用的`init(name: String)`替换成子类`RecipeIngredient`重写过后的便利构造器。
``` swift
class ShoppingListItem: RecipeIngredient {
    var purchased = false
    var description: String {
    var output = "\(quantity) x \(name.lowercaseString)"
        output += purchased ? " ✔" : " ✘"
        return output
    }
}

var breakfastList = [
    ShoppingListItem(),
    ShoppingListItem(name: "Bacon"),
    ShoppingListItem(name: "Eggs", quantity: 6),
]
breakfastList[0].name = "Orange juice"
breakfastList[0].purchased = true
for item in breakfastList {
    println(item.description)
}
// 1 x orange juice ✔
// 1 x bacon ✘
// 6 x eggs ✘
```
类`ShoppingListItem`继承类`RecipeIngredient`。
因为子类的新属性`purchased`有默认值，而且自己没有定义任何构造器，所以继承了父类的所有指定构造器。
这时就满足上面提到继承父类构造器的第二个条件：子类提供了所有父类指定构造器的实现。
所以子类也继承了父类的所有便利构造器。
类`shoppingListItem`就继承了`init()`，`init(name: String)`和`init(name: String, quantity: Int)`三种构造器。

## 可失败构造器
构造过程中可能因为传入无效参数值，缺少资源，不满足必要条件等原因构造失败的构造器，称为可失败构造器。

### 基本语法
可失败构造器在`init`关键字后面加上`?`，即`init?`。
并在失败的情况下加上`return nil`使构造器返回`nil`，非可失败构造器中不能使用`return`返回值。
可失败构造器的参数名和参数类型不能与其他非可失败构造器完全相同。
``` swift
struct Animal {
    let species: String
    init?(species: String) {
        if species.isEmpty { return nil }
        self.species = species
    }
}

let someCreature = Animal(species: "Giraffe")
// someCreature 的类型是 Animal? 而不是 Animal

if let giraffe = someCreature {
    println("An animal was initialized with a species of \(giraffe.species)")
}
// 打印 "An animal was initialized with a species of Giraffe"

let anonymousCreature = Animal(species: "")
// anonymousCreature 的类型是 Animal?, 而不是 Animal

if anonymousCreature == nil {
    println("The anonymous creature could not be initialized")
}
// 打印 "The anonymous creature could not be initialized"
```

### 枚举类型的可失败构造器
``` swift
enum TemperatureUnit {
    case Kelvin, Celsius, Fahrenheit
    init?(symbol: Character) {
        switch symbol {
        case "K":
            self = .Kelvin
        case "C":
            self = .Celsius
        case "F":
            self = .Fahrenheit
        default:
            return nil
        }
    }
}

let fahrenheitUnit = TemperatureUnit(symbol: "F")
if fahrenheitUnit != nil {
    println("This is a defined temperature unit, so initialization succeeded.")
}
// 打印 "This is a defined temperature unit, so initialization succeeded."

let unknownUnit = TemperatureUnit(symbol: "X")
if unknownUnit == nil {
    println("This is not a defined temperature unit, so initialization failed.")
}
// 打印 "This is not a defined temperature unit, so initialization failed."
```
当参数值不能与任意一枚举成员相匹配时，该枚举类型的构建过程失败。

带原始值的枚举类型会自带一个可失败构造器`init?(rawValue: )`，`rawValue`是一个默认参数，和枚举类型的原始值类型一致。
如果该参数的值能和枚举类型成员所带的原始值匹配，则构建器构造一个带此原始值的枚举成员，否则构造失败。
上面的例子可以重写为：
``` swift
enum TemperatureUnit: Character {
    case Kelvin = "K", Celsius = "C", Fahrenheit = "F"
}

let fahrenheitUnit = TemperatureUnit(rawValue: "F")
if fahrenheitUnit != nil {
    println("This is a defined temperature unit, so initialization succeeded.")
}
// prints "This is a defined temperature unit, so initialization succeeded."

let unknownUnit = TemperatureUnit(rawValue: "X")
if unknownUnit == nil {
    println("This is not a defined temperature unit, so initialization failed.")
}
// prints "This is not a defined temperature unit, so initialization failed."
```

### 类的可失败构造器
类的可失败构造器只能在所有类属性被初始化和所有类之间的构造代理之间的代理调用发生完后触发失败行为。
而值类型的可失败构造器可以随时随地触发。
``` swift
class Product {
    let name: String!
    init?(name: String) {
        self.name = name
        if name.isEmpty { return nil }
    }
}

if let bowTie = Product(name: "bow tie") {
    // 不需要检查 bowTie.name == nil
    println("The product's name is \(bowTie.name)")
}
// 打印 "The product's name is bow tie"
```
类`Product`的可失败构造器必须建立在`name`被赋值的情况下。
所以`name`被声明为隐式解析可选类型（`String!`）保证触发失败条件时，`name`一定有值。
类`Prodcut`构建成功时，`name`一定有一个非`nil`值，可以直接访问`name`。

### 可失败构造器的代理
可失败构造器的代理规则和构造器基本一致，只是一旦触发构造失败，整个构造过程就会被立即终止。
可失败构造器可以在同一类中代理调用其他非可失败构造器，这样可以为已有的构造器添加构造失败的条件。
``` swift
class Product {
    let name: String!
    init?(name: String) {
        if name.isEmpty { return nil }
        self.name = name
    }
}

class CartItem: Product {
    let quantity: Int!
    init?(name: String, quantity: Int) {
        super.init(name: name)
        if quantity < 1 { return nil }
        self.quantity = quantity
    }
}

if let twoSocks = CartItem(name: "sock", quantity: 2) {
    println("Item: \(twoSocks.name), quantity: \(twoSocks.quantity)")
}
// 打印 "Item: sock, quantity: 2"

if let zeroShirts = CartItem(name: "shirt", quantity: 0) {
    println("Item: \(zeroShirts.name), quantity: \(zeroShirts.quantity)")
} else {
    println("Unable to initialize zero shirts")
}
// 打印 "Unable to initialize zero shirts"

if let oneUnnamed = CartItem(name: "", quantity: 1) {
    println("Item: \(oneUnnamed.name), quantity: \(oneUnnamed.quantity)")
} else {
    println("Unable to initialize one unnamed product")
}
// 打印 "Unable to initialize one unnamed product"
```
第18行：构造成功。
第23行：`quantiry`的值小于`1`，不满足条件，构造失败。
第30行：`name`为空，父类`Product`可失败构造器触发构造失败，整个构造过程停止并失败。

### 可失败构造器的重写
父类的可失败构造器可以被子类的可失败构造器或者非可失败构造器重写。
但是父类的非可失败构造器不可以被子类的可失败构造器重写。
如果用非可失败构造器重写可失败构造器时，不再向上代理父类的可失败构造器，非可失败构造器不不会代理调用可失败构造器。
``` swift
class Document {
    var name: String?
    // 该构造器构建了一个name属性值为nil的document对象
    init() {}
    // 该构造器构建了一个name属性值为非空字符串的document对象
    init?(name: String) {
        if name.isEmpty { return nil }
        self.name = name
    }
}

class AutomaticallyNamedDocument: Document {
    override init() {
        super.init()
        self.name = "[Untitled]"
    }
    override init(name: String) {
        super.init()
        if name.isEmpty {
            self.name = "[Untitled]"
        } else {
            self.name = name
        }
    }
}
```

### 隐私解析可选类型的可失败构造器
`init!`同`init?`一样都是可失败构造器，该可失败构造器就会构造一个特定类型的隐私解析可选类型的对象。
`init?`和`init!`可以相互代理调用，相互重写。
`init`也可以代理调用`init!`，但这会触发一个断言：`init!`是否会触发构造失败。

### 必要构造器
在类的构造器前添加`required`关键字表示该类的子类都必须实现该构造器。
子类重写父类的`required`必要构造器时，也要加上`required`关键字，也是必要构造器。
覆盖基类的必要构造器时，不需要添加`override`关键字。
``` swift
class SomeClass {
    required init() {
        // 在这里添加该必要构造器的实现代码
    }
}

class SomeSubclass: SomeClass {
    required init() {
        // 在这里添加子类必要构造器的实现代码
    }
}
```
不一定需要显示的实现父类的必要构造器，只要满足父类的必要构造器需求即可。

## 闭包设置属性默认值
闭包可以用来为属性提供定制的默认值，返回和属性类型相同类型的默认值。
在闭包中不能使用其他属性，不能访问其他实例方法，不能使用`self`属性。
``` swift
class SomeClass {
    let someProperty: SomeType = {
        // 在这个闭包中给 someProperty 创建一个默认值
        // someValue 必须和 SomeType 类型相同
        return someValue
        }()
}
```
闭包后面接`()`表示闭包立刻执行，否则会把闭包赋值给`someProperty`。
``` swift
struct Checkerboard {
    let boardColors: [Bool] = {
        var temporaryBoard = [Bool]()
        var isBlack = false
        for i in 1...10 {
            for j in 1...10 {
                temporaryBoard.append(isBlack)
                isBlack = !isBlack
            }
            isBlack = !isBlack
        }
        return temporaryBoard
        }()
    func squareIsBlackAtRow(row: Int, column: Int) -> Bool {
        return boardColors[(row * 10) + column]
    }
}

let board = Checkerboard()
println(board.squareIsBlackAtRow(0, column: 1))
// 输出 "true"
println(board.squareIsBlackAtRow(9, column: 9))
// 输出 "false"
```
这里的闭包把类`Checkerboard`的布尔型数组`boardColors`初始化为`true`和`false`交替的数组，可以用来标识国际象棋的棋盘。

# 析构过程
Swift会自动释放不再需要的实例以释放资源。如果我们需要进行一些额外的清理，就需要使用析构函数。
每个类最多只能有一个析构函数。
析构函数使用关键字`deinit`，不带任何参数，在写法上不带括号：
``` swift
class ClassName {
    deinit { 
	//some action
    }
}
```
析构函数是在实例释放前被自动调用，不允许自己主动调用。
子类的析构函数先调用，父类的析构函数后调用。子类没有提供析构函数，也会调用父类的析构函数。

# 嵌套类型
枚举，类和结构体可以想换嵌套，将需要嵌套的类型定义写在被嵌套类型的区域{}内，可以实现多级嵌套。

``` swift
struct BlackjackCard {
    // 嵌套定义枚举型Suit
    enum Suit: Character {
       case Spades = "♠", Hearts = "♡", Diamonds = "♢", Clubs = "♣"
    }

    // 嵌套定义枚举型Rank
    enum Rank: Int {
       case Two = 2, Three, Four, Five, Six, Seven, Eight, Nine, Ten
       case Jack, Queen, King, Ace
       struct Values {
           let first: Int, second: Int?
       }
       var values: Values {
        switch self {
        case .Ace:
            return Values(first: 1, second: 11)
        case .Jack, .Queen, .King:
            return Values(first: 10, second: nil)
        default:
            return Values(first: self.toRaw(), second: nil)
            }
       }
    }

    // BlackjackCard 的属性和方法
    let rank: Rank, suit: Suit
    var description: String {
    var output = "suit is \(suit.toRaw()),"
        output += " value is \(rank.values.first)"
        if let second = rank.values.second {
            output += " or \(second)"
        }
        return output
    }
}
```
结构体`BlackjackCard`用来存储“二十一点游戏”中的扑克牌，嵌套了枚举类型`Suit`表示花色，嵌套了枚举类型`Rank`表示点数。而且`Rank`中又定义了结构体`Values`准确描述牌的大小：数字牌表示本身数字的大小，`Ace`表示1或者11，`Jack`，`Queen`和`King`表示10。
结构体有默认的成员构造函数，这里的默认构造函数为：
``` swift
let theAceOfSpades = BlackjackCard(rank: .Ace, suit: .Spades)
println("theAceOfSpades: \(theAceOfSpades.description)")
// 打印出 "theAceOfSpades: suit is ♠, value is 1 or 11"

let heartsSymbol = BlackjackCard.Suit.Hearts.toRaw()
// 红心的符号 为 "♡"
```
