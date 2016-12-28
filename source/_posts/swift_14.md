title: "Swift基础入门(14)：权限控制"
date: 2015-07-29 11:28:32
tags:
- swift
categories: swift
toc: false
---

本篇介绍Swift的基础知识：权限控制，包括公开访问（_pubilc_），内部访问（_internal_），私有访问（_private_）三种访问方式控制实体访问的权限。

<!--more-->
**Title: [Swift基础入门(14)：权限控制](https://aidaizyy.github.io/swift_14)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2015-07-29](http://aidaizyy.github.io)**

# 权限控制
我们可以给基本类型、常量变量、函数、类、结构体、枚举、属性、方法、下标等等设置访问级别确定访问权限。
- public：公开访问，实体能够被当前模块（_module_）中的所有源文件访问，也可以被其他引用了该模块的另一个模块中的所有源文件访问。
- internal：内部访问，实体能够被当前模块中的所有源文件访问，但是不可以被其他引用了该模块的另一个模块中的源文件访问。
- private：私有访问，实体只能在当前源文件中访问，不能被其他任何源文件访问。

默认的权限为`internal`，`public`和`private`必须指定。

对于属性，可以设置取值权限比赋值权限更加开放，即getter的权限比setter高。
比如下面的例子，用`private(set)`把属性的setter权限设置为私有访问，而getter的权限仍然为默认的`internal`内部访问。
``` swift
    public class ListItem {

    // ListItem这个类，有两个公开的属性
    public var text: String
    public var isComplete: Bool

    // 下面的代码表示把变量UUID的赋值权限设为private，对整个app可读，但值只能在本文件里写入
    private(set) var UUID: NSUUID

    public init(text: String, completed: Bool, UUID: NSUUID) {
        self.text = text
        self.isComplete = completed
        self.UUID = UUID
    }

    // 这段没有特别标记权限，因此属于默认的internal级别。在框架目标内可用，但对于其他目标不可用
    func refreshIdentity() {
        self.UUID = NSUUID()
    }

    public override func isEqual(object: AnyObject?) -> Bool {
        if let item = object as? ListItem {
            return self.UUID == item.UUID
        }
        return false
        }
    }
```

** 转载请注明原作者和出处。**
> 如果觉得这篇文章对您有帮助或启发，请随意打赏~
<p> <img src="http://7xivk7.com1.z0.glb.clouddn.com/paycode01.jpg" width = "250" align = "left" > <img src="http://7xivk7.com1.z0.glb.clouddn.com/paycode02.jpg" width = "250" align = "left" /> </p>
