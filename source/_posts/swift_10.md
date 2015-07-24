title: "Swift基础入门(10)：类型转换"
date: 2015-07-24 15:23:06
tags:
- swift
categories: swift
toc: true
---

本篇介绍Swift的基础知识：类型的检查和转换。

<!--more-->
**Title: [Swift基础入门(10)：类型转换](https://aidaizyy.github.io/swift_10)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2015-07-24](http://aidaizyy.github.io)**

# 类型转换

## 数值型类型转换
- Int，Double，Float：
`Int16`与`Int8`不能直接相加，需要通过`Int16(Int8)`转换。同样，`Int8`，`UInt8`，`Int16`，`UInt16`，`Int32`，`UInt32`，`Int64`，`UInt64`都可以互相转换。
`Double`与`Int`也不能相加，也需要通过`Double(Int)`转换，如果只需要整数部分，也可以通过`Int(Double)`转换。同样，`Float(Int)`，`Int(Float)`，`Double(Float)`，`Float(Double)`都可以互相转换

- String，Int：
String->Int：`String.toInt()`函数可以把`String`转换成可选类型`Int?`，因为`String`中不一定能转换成`Int`，所以得到可选类型。
Int->String：`String(Int)`函数可以把`Int`转换成`String`。

## 类型检查和向下转换
类型检查用`is`操作符检查一个实例是否属于特定类型，返回`true`或者`false`。
基类类型用`as?`或者`as!`操作符转换成子类类型。因为转换可能失败，所以使用`as?`返回可选类型的子类类型；如果确定转换一定成功，可以使用`as!`强制返回非可选类型的子类类型。
``` swift
class MediaItem {
    var name: String
    init(name: String) {
        self.name = name
    }
}

class Movie: MediaItem {
    var director: String
    init(name: String, director: String) {
        self.director = director
        super.init(name: name)
    }
}

class Song: MediaItem {
    var artist: String
    init(name: String, artist: String) {
        self.artist = artist
        super.init(name: name)
    }
}

let library = [
    Movie(name: "Casablanca", director: "Michael Curtiz"),
    Song(name: "Blue Suede Shoes", artist: "Elvis Presley"),
    Movie(name: "Citizen Kane", director: "Orson Welles"),
    Song(name: "The One And Only", artist: "Chesney Hawkes"),
    Song(name: "Never Gonna Give You Up", artist: "Rick Astley")
]
// the type of "library" is inferred to be [MediaItem]

var movieCount = 0
var songCount = 0
for item in library {
    if item is Movie {
        ++movieCount
    } else if item is Song {
        ++songCount
    }
}
println("Media library contains \(movieCount) movies and \(songCount) songs")
// prints "Media library contains 2 movies and 3 songs"

for item in library {
    if let movie = item as? Movie {
        println("Movie: '\(movie.name)', dir. \(movie.director)")
    } else if let song = item as? Song {
        println("Song: '\(song.name)', by \(song.artist)")
    }
}
// Movie: 'Casablanca', dir. Michael Curtiz
// Song: 'Blue Suede Shoes', by Elvis Presley
// Movie: 'Citizen Kane', dir. Orson Welles
// Song: 'The One And Only', by Chesney Hawkes
// Song: 'Never Gonna Give You Up', by Rick Astley
```
类`Movie`和类`Song`都是继承自类`MediaItem`。
数组`Libray`自动判断类型为`MediaItem`，存入了两个`Movie`实例和三个`Song`实例。在`for-in`循环中遍历出来的都是基类类型，但它实际上存储的是子类类型。
第33-43行：`item is Movie`和`item is Song`判断`item`实际存储的值是不是子类类型，是的话在相应数量的记录上加1，最后输出各子类类型的数组元素的个数。
第45-56行：`item as? Movie`和`item as? Song`将`item`强制转换成子类类型，如果实际存储的不相符返回`nil`，实际存储的相符返回相应的子类类型，并打印相应信息。

## Any和AnyObject类型转换
- Any：任意类型，包括方法类型。
- AnyObject：任意class类型。
``` swift
let someObjects: [AnyObject] = [
    Movie(name: "2001: A Space Odyssey", director: "Stanley Kubrick"),
    Movie(name: "Moon", director: "Duncan Jones"),
    Movie(name: "Alien", director: "Ridley Scott")
]

var things = [Any]()
things.append(0)
things.append(0.0)
things.append("hello")
```
`someObject`是一个很多class类型组成的混合class类型数组，所以用`AnyObject`。
`things`是一个很多类型组成的混合类型数组，所以用`Any`。
它们的访问都可以通过遍历，然后使用`is`判断或者`as`转换。

> 在`switch`的`case`语句中，使用`as`而不是`as?`。因为`case`语句中类型的检查和转换总是安全的。

