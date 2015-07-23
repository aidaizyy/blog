title: "Swift基础入门(5)：函数和闭包"
date: 2015-07-16 17:11:53
tags:
- swift
categories: swift
toc: true
---

本篇介绍Swift的基础知识：函数和闭包（Closure）。

<!--more-->
**Title: [Swift基础入门(5)：函数和闭包](https://aidaizyy.github.io/swift_5)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2015-07-22](http://aidaizyy.github.io)**

# 函数
## 函数原型
`func functionName(parameters) -> returnType { statements }`
``` swift
func halfOpenRangeLength(start: Int, end: Int) -> Int {
    return end - start
}
println(halfOpenRangeLength(1, 10))
// prints "9"
```
函数前必须加标识符`func`，函数名`halfOpenRangeLength`需要传入两个参数`start`和`end`，都是`Int`类型，返回`Int`类型。
可以没有参数，也可以没有返回值，则写作`func halfOpenRangeLength() { statements }`，`statements`中不带`return`语句。

函数可以有多个返回值，用元组表示返回值。
``` swift
func count(string: String) -> (vowels: Int, consonants: Int, others: Int) {
    var vowels = 0, consonants = 0, others = 0
    for character in string {
        switch String(character).lowercaseString {
        case "a", "e", "i", "o", "u":
            ++vowels
        case "b", "c", "d", "f", "g", "h", "j", "k", "l", "m",
          "n", "p", "q", "r", "s", "t", "v", "w", "x", "y", "z":
            ++consonants
        default:
            ++others
        }
    }
    return (vowels, consonants, others)
}

let total = count("some arbitrary string!")
println("\(total.vowels) vowels and \(total.consonants) consonants")
// prints "6 vowels and 13 consonants"
```
`func count(String) -> (Int, Int, Int)`函数，传入一个`String`值，返回一个带3个`Int`值的元组。
返回的元组成员不需要再命名，因为在函数定义时已经命名了返回元组成员的名称。
当然也可以不命名，返回类型写作`(Int, Int, Int)`的形式，用`total.0`，`total.1`和`total.2`去获取元组`total`的第1个，第2个和第3个成员的值。

## 函数参数

### 外部参数名
函数参数的名称作为局部参数名，只能在函数中使用。定义外部参数名，可以在函数外部使用帮助函数参数的意图清晰，`func functionName(externalParameterName localParameterName: dataType) { statements }`。
注意：但是一旦定义了外部参数名，在函数调用时就**必须使用**。
外部参数名和局部参数名如果一致，可在局部参数名前加`#`表示，`func functionName(#parameterName: dataType) { statements }`。
``` swift
func containsCharacter(str string: String, #characterToFind: Character) -> Bool {
    for character in string {
        if character == characterToFind {
            return true
        }
    }
    return false
}
let containsAVee = containsCharacter(str: "aardvark", characterToFind: "v")
// containsAVee equals true, because "aardvark" contains a "v”
```

### 默认参数值
函数参数可以定义默认值，但必须在函数参数列表的最后。调用时，如果不指定参数的值，则使用默认值。
定义了默认值的函数参数会自动提供外部参数名，和局部参数名一样，也可以自己提供，如果不使用默认值，则必须在调用时使用外部参数名。
``` swift
func join(s1: String, s2: String, joiner: String = " ", flag: String = "!") -> String {
    return s1 + joiner + s2 + flag
}

join("hello", "world", joiner: "-")
// returns "hello-world!"
```
第3个参数`joiner`和第四个参数`flag`都是提供了默认参数值，自动提供了外部参数名`joiner`和`flag`，与局部参数名一致。
`joiner`提供了值，则必须使用参数名`joiner: "-"`。
`flag`使用默认值，则不需要在调用时出现。

### 可变参数
可变参数（_variadic parameter_）表示不确定数量的输入参数，在参数后加`...`表示。一个函数最多只能有一个可变参数， 且必须是参数列表的最后一个参数。
``` swift
func arithmeticMean(numbers: Double...) -> Double {
    var total: Double = 0
    for number in numbers {
        total += number
    }
    return total / Double(numbers.count)
}
arithmeticMean(1, 2, 3, 4, 5)
// returns 3.0, which is the arithmetic mean of these five numbers
arithmeticMean(3, 8, 19)
// returns 10.0, which is the arithmetic mean of these three numbers
```

### 变量参数
Swift的函数参数采用值拷贝传递，传递进去的参数是不能进行修改的，如果我们需要，可以定义变量参数。在参数前加`var`定义变量参数。
``` swift
func appendCharacter(var string: String, flag: Character) -> String {
    string.append(flag)
    return string
}
let originalString = "hello"
let paddedString = appendCharacter(originalString, "!")
// paddedString is equal to "hello!"
```
`string`在函数内被修改了，但是作为局部变量，只能在函数内部使用。

### 输入输出参数
Swift的函数参数不能被修改，使用变量参数修改后也不能传递到外部，采用输入输出参数可以解决这个问题。
在参数前加`inout`定义输入输出参数。
- 函数调用时，输入输出参数只能传入变量
- 输入输出参数不能有默认参数值。
- 输入输出参数不能是可变参数。
``` swift
func appendCharacter(inout string: String, flag: Character) {
    string.append(flag)
}
var originalString = "hello"
appendCharacter(&originalString, "!")
// originalString is equal to "hello!"
```
`appendCharacter`函数传入参数时，在输入输出参数前必须加`&`前缀。

## 函数类型

函数类型和其他类型一样，可以定义并赋值，如：
``` swift
func addTwoInts(a: Int, b: Int) -> Int {
    return a + b
}

var mathFunciton1: (Int, Int) -> Int =addTwoInts

var mathFunciton2 = addTwoInts		//通过赋值自动判断mathFunction类型为函数类型

println("Result: \(mathFunction1(2, 3))")
// prints "Result: 5"
println("Result: \(mathFunction2(2, 3))")
// prints "Result: 5"
```

同样函数类型可以作为函数的参数类型和返回类型，形式如`func printMathResult(mathFunction: (Int, Int) -> Int, a: Int, b: Int)`和`func chooseStepFunction(backwards: Bool) -> (Int) -> Int`。前者的一个参数为`mathFunction: (Int, Int) -> Int`，后者的返回`(Int) -> Int`，都没有`func`关键字。

函数也支持嵌套函数，在函数A内部定义的函数B只能在函数A内调用。

# 闭包

## 闭包表达式
闭包指自包含的函数代码块，可以在代码中被传递和使用。
函数就是特殊的闭包。
闭包的一般形式：`{ (parameters) -> returnType in statements }
和函数不同的是，用`in`替代了原本函数的大括号，并在最外层加上大括号。
``` swift
func backwards(s1: String, s2: String) -> Bool {
    return s1 > s2
}
var reversed = sorted(names, backwards)
// reversed 为 ["Ewa", "Daniella", "Chris", "Barry", "Alex"]
```
`sorted`函数需要两个参数，第一个参数是需要排序的数组，第二个参数是确定排序顺序的闭包函数，传入与数组类型相同的两个值，并返回`Bool`值。如果第二个参数返回`true`则两个数组元素顺序不变；如果第二个参数返回`false`则两个数组元素顺序相反。
所以闭包函数中定义`return s1 > s2`，如果`s1`大于`s2`顺序不变，如果`s1`不大于`s2`则交换`s1`和`s2`的顺序，使值大的元素排在数组的前列，也就是逆序排列。
就上面的代码改为闭包表达式的形式为：
``` swift
reversed = sorted(names, { (s1: String, s2: String) -> Bool in
    return s1 > s2
})
```
用闭包表达式代替了闭包函数，`in`替换函数的大括号，并在外层添加大括号。

闭包表达式的参数类型由第一个参数数组元素的类型决定，返回类型确定为`Bool`型，创建闭包时可以省略已知的信息：
``` swift
reversed = sorted(names, { s1, s2 in return s1 > s2 })
```
闭包表达式中，如果只有单行表达式，比如`return s1 > s2`一行，可以省略`return`关键字：
``` swift
reversed = sorted(names, { s1, s2 in s1 > s2 })
```
闭包表达式的参数名称可以缩写成$0，$1，$2等，来顺序调用闭包参数：
``` swift
reversed = sorted(names, { $0 > $1 })
```
另外，还可以用运算符函数（operator function）使闭包表达式更简短。因为`>`的定义就是接收两个参数，并返回`Bool`类型值，所以可以写：
``` swift
reversed = sorted(names, >)
```
尾随闭包（trailing closure）：如果闭包表达是是函数的最后一个参数，可以把闭包放到函数的小括号后面，增强可读性：
``` swift
reversed = sorted(names, { $0 > $1 })	//闭包表达式

reversed = sorted(names) { $0 > $1 }	//尾随闭包
```
如果闭包很长，尾随闭包就会非常有用。
如果函数中只有闭包一个参数，则可以省略小括号，写成`reversed = sorted { $0 < $1 }`的形式。

## 嵌套函数
嵌套函数是最简单的闭包形式。嵌套函数可以捕获外部函数的参数和定义的常量变量。
``` swift
func makeIncrementor(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementor() -> Int {
        runningTotal += amount
        return runningTotal
    }
    return incrementor
}
```
`incrementor`函数调用的`amount`是外部函数的参数，捕获并存储了副本；`runningTotal`会被修改，所以不可以是副本，而是捕获了一个引用，就算外部函数结束都不会消失。Swift会自动决定捕获引用还是副本。
``` swift
let incrementByTen = makeIncrementor(forIncrement: 10)

incrementByTen()
// 返回的值为10
incrementByTen()
// 返回的值为20
incrementByTen()
// 返回的值为30

let incrementBySeven = makeIncrementor(forIncrement: 7)
incrementBySeven()
// 返回的值为7
incrementByTen()
// 返回的值为40
```
`incrementByTen`创建时，`runningTotal`也创建了，每调用一次函数其值就会增加10。
`incremetnBySeven`创建时，一个新的`runningTotal`也创建了，每调用一次函数其值就会增加7。这个变量和`incrementByTen`中的变量没有任何关系，互不干扰。

注意：无论是函数还是闭包，在赋值给常量或变量时都是**引用拷贝**，指向的是同一个函数/闭包对象。
