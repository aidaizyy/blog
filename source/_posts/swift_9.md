title: "Swift基础入门(9)：可选链和自动引用计数"
date: 2015-07-23 15:43:48
tags:
categories: 
toc: true
---

本篇介绍Swift的基础知识：通过可选链（_optional chaining_）调用属性，方法和下标脚本；自动引用计数（_automatic reference counting_）的工作机制；循环强引用的解决方案。

<!--more-->
**Title: [Swift基础入门(9)：可选链自动引用计数](https://aidaizyy.github.io/swift_9)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2015-07-22](http://aidaizyy.github.io)**

# 可选链
如果请求和调用属性，方法和下标脚本的目标可能为空`nil`，这样的多次请求或调用就可以被链接起来，称为可选链。
如果任何一个节点为空`nil`，整个可选链失效。
``` swift
class Person { //人
    var residence: Residence? //每个人可能有住所
}

class Residence { //住所
    var id: String? //住所可能有ID
    var address: Address? //住所可能有地址
    var rooms = [Room]() //住所的房间
    var numberOfRooms: Int { //返回住所的房间数量
        return rooms.count
    }
    subscript(i: Int) -> Room { //下标地址访问住所的一个房间
        return rooms[i]
    }
    func printNumberOfRooms() { //打印房间的数量
        println("The number of rooms is \(numberOfRooms)")
    }
    func getId() -> String? { //返回住所的ID
        if id != nil {
            return id
        } else {
            return nil
        }
    }
}

class Room { //房间
    let name: String //房间的名字
    init(name: String) { self.name = name } //房间的构造器
}

class Address { //地址
    var street: String? //地址可能有街道
}

let john = Person()

//属性
if let roomCount = john.residence?.numberOfRooms {
    println("John's residence has \(roomCount) room(s).")
} else {
    println("Unable to retrieve the number of rooms.")
}
// 打印 "Unable to retrieve the number of rooms。

//无返回值方法
if john.residence?.printNumberOfRooms() != nil{
    println("It was possible to print the number of rooms.")
} else {
    println("It was not possible to print the number of rooms.")
}
// 打印 "It was not possible to print the number of rooms."。

//有返回值方法
if let buildingId = john.residence?.getId() {
    println("John's building identifier is \(buildingId).")
} else {
    println("Unable to retrieve the ID of building.")
}
// 打印 "John's building identifier is The Larches."。

//下标脚本
if let firstRoomName = john.residence?[0].name {
    println("The first room name is \(firstRoomName).")
} else {
    println("Unable to retrieve the first room name.")
}
// 打印 "Unable to retrieve the first room name."。

//多层链接
if let johnsStreet = john.residence?.address?.street {
    println("John's street name is \(johnsStreet).")
} else {
    println("Unable to retrieve the address.")
}
// 打印 "Unable to retrieve the address.”。
```
创建了`Person`的实例`john`，`john`中包含类`Residence`的实例`resindence`。
`resindence`包含了可选类型的属性和方法，值可能为`nil`，所以访问它的属性，方法和下标脚注时都应该加上`?`。
可选链中只要有一个节点为可选类型，可选链的结果就一定为可选类型。
第39行，`john.residence?.numberOfRooms`的结果类型为`Int?`。
第47行，`john.residence?.printNumberOfRooms()`的结果类型为`void?`。这里不能直接用函数结构作为布尔型去判断，而是与`nil`比较。
第55行，`john.residence?.getId()`的结果类型为`String?`。
第63行，`john.residence?[0].name`的结果类型为`String?`。这里的`?`放在`[0]`前，因为确保数组有值，才能通过下标脚注去访问。
第71行，`john.reidence?.address?.street`的结果类型为`String?`。多层的可选链接链接到一起，`residence`和`address`都是可选类型，所以使用了两个`?`。如果给`john.residence.address`中的`address`分配实例，应该写作`john.residence!.address`，强制解析确保`residence`有值，才能对其中的`address`分配实例。

# 自动引用计数
自动引用计数（_ARC_）跟踪和管理内存，会自动释放不再使用的实例占用的内存。
ARC会跟踪和计算每一个实例被多少属性，常量和变量引用，这样的引用称为对实例的强引用。不存在强引用，实例会被销毁，否则实例会被保留。

## 类实例的循环强引用
两个类中相互引用，相互保持对方的强引用，这样无法销毁，形成了循环强引用。

### 强引用
``` swift
class Person {
    let name: String
    init(name: String) { self.name = name }
    var apartment: Apartment?
    deinit { println("\(name) is being deinitialized") }dd
}

class Apartment {
    let number: Int
    init(number: Int) { self.number = number }
    var tenant: Person?
    deinit { println("Apartment #\(number) is being deinitialized") }
}

var john: Person?
var number73: Apartment?

john = Person(name: "John Appleseed")
number73 = Apartment(number: 73)

john!.apartment = number73
number73!.tenant = john

john = nil
number73 = nil
```
上面的代码展示了类实例的循环强引用。
类`Person`中有属性是`Apartment`类型，类`Apartment`中有属性是`Person`类型。
声明了实例`john`和实例`number73`，并赋值。
最后两行把两个实例都设为`nil`，但是析构函数并没有被调用，因为两个实例还有循环强引用联系，并没有自动销魂，而且造成了内存泄露。

### 弱引用
为了解决循环强引用问题，有两种办法：弱引用（_weak reference_）和无主引用（_unowned reference_）。
一个实例对另一个实例弱引用或者无主引用，不产生强引用，反过来，另一个实例对一个实例强引用，这样能够相互引用而不产生循环强引用。如果实例的值可能为`nil`使用弱引用；如果实例的值不可能为`nil`使用无主引用。

声明时在属性或常量变量前加上`weak`关键字表示弱引用。
两个实例的值都可能为`nil`，使用弱引用。
``` swift
lass Person {
    let name: String
    init(name: String) { self.name = name }
    var apartment: Apartment?
    deinit { println("\(name) is being deinitialized") }
}

class Apartment {
    let number: Int
    init(number: Int) { self.number = number }
    weak var tenant: Person?
    deinit { println("Apartment #\(number) is being deinitialized") }
}

var john: Person?
var number73: Apartment?

john = Person(name: "John Appleseed")
number73 = Apartment(number: 73)

john!.apartment = number73
number73!.tenant = john

john = nil
// prints "John Appleseed is being deinitialized"
number73 = nil
// prints "Apartment #73 is being deinitialized"
```
上面的代码和循环强引用代码基本一致，只是在类`Apartment`的类型为`Person?`的属性`tenant`前加上了`weak。因为`Person?`是可选类型，`tenant`值可能为`nil`，所以使用弱引用。
最后一句，赋值`nil`给`john`后，因为`number73`对`john`不是强引用，`john`这时没有强引用，可以销毁，调用了析构函数。`john`销毁后，`number73`没有强引用，也可以被销毁。
如果把`number73 = nil`和`john = nil`语句顺序交换，打印顺序不会变，因为没有强引用的`john`一定是先销毁。

### 无主引用
声明时在属性或常量变量前加上`unowned`关键字表示无主引用。
- 一个实例的值可能为`nil`，另一个实例的值不可能为`nil`。
``` swift
class Customer {
    let name: String
    var card: CreditCard?
    init(name: String) {
        self.name = name
    }
    deinit { println("\(name) is being deinitialized") }
}

class CreditCard {
    let number: Int
    unowned let customer: Customer
    init(number: Int, customer: Customer) {
        self.number = number
        self.customer = customer
    }
    deinit { println("Card #\(number) is being deinitialized") }
}

var john: Customer?

john = Customer(name: "John Appleseed")
john!.card = CreditCard(number: 1234_5678_9012_3456, customer: john!)

john = nil
// prints "John Appleseed is being deinitialized"
// prints "Card #1234567890123456 is being deinitialized"
```
类`CreditCard`的类型为`Customer`的属性`customer`前加上了`unowned`。因为`customer`在构造函数中就会赋予初始值，值不会为`nil`，所以使用无主引用。
最后一句，赋值`nil`给`john`后，因为类`CreditCard`的实例对`john`不是强引用，`john`这时没有强引用，可以销毁，调用了析构函数。`john`销毁后，类`Creditcard`的实例也没有强引用了，跟着被销毁了。
- 两个实例的值都不可能为`nil`。
这种场景一个类使用无主属性，另一个类使用隐式解析可选类型。
``` swift
class Country {
    let name: String
    let capitalCity: City!
    init(name: String, capitalName: String) {
        self.name = name
        self.capitalCity = City(name: capitalName, country: self)
    }
}

class City {
    let name: String
    unowned let country: Country
    init(name: String, country: Country) {
        self.name = name
        self.country = country
    }
}

var country = Country(name: "Canada", capitalName: "Ottawa")
println("\(country.name)'s capital city is called \(country.capitalCity.name)")
// prints "Canada's capital city is called Ottawa"
```
类`City`的类型为`Country`的属性`country`前加上了`unowned`表示无主属性。
类`Country`的类型为`City!`的属性`capitalCity`在类型后加上了`!`表示隐式解析可选类型。
类`Country`的实例的`country`创建时，调用构造器，为`name`和`capiptalCity`赋值。因为`self`属性必须在构造的第二阶段使用，也就是类中所有存储型属性全部有初始值之后才能使用。如果`Country`的值可以为`nil`，先给`capitalCity`赋值`nil`就可以调用构造器为`capitalCity`赋予新值。但是这里不可以赋值`nil`，所以加上了`!`隐私解析可选类型，默认初始值为`nil`，可以调用构造器赋予新值。
属性`capitalCity`在调用时可以直接使用，不再需要加`!`访问。

## 闭包的循环强引用
除了两个类实例的循环强引用，类实例和闭包也可能引起循环强引用，比如把闭包赋值给类的一个属性，而闭包中又通过`self`访问类的一个属性，这就引起了循环强引用，实例不会被销毁。

### 闭包占用列表
闭包占用列表（_closuer capture list_）可以解决闭包引起的循环强引用问题。
> 闭包调用类属性，必须加上`self.`，不能直接通过属性名调用。
``` swift
lazy var someClosure: (Int, String) -> String = {
    [unowned self] (index: Int, stringToProcess: String) -> String in
    // closure body goes here
}
```
定义占用列表，使用`weak`或`unowned`，视值是否能为`nil`而定。
``` swift
class HTMLElement {

    let name: String
    let text: String?

    lazy var asHTML: () -> String = {
        [unowned self] in
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }

    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }

    deinit {
        println("\(name) is being deinitialized")
    }

}

var paragraph: HTMLElement? = HTMLElement(name: "p", text: "hello, world")
println(paragraph!.asHTML())
// prints "<p>hello, world</p>"

paragraph = nil
// prints "p is being deinitialized"
```
上面的例子中，闭包的指定参数列表和返回类型可以通过上下文推断，所以省略。`in`放在占用列表之后。
这里使用了无主引用，闭包通过`unowned self`对类`HTMLELement`无主引用。
最后一句，赋值`nil`给`paragraph`，没有了闭包对它的强引用，可以销毁并调用析构函数。
