title: 容量失效（capacity miss）与冲突失效（conflict miss）的区别
date: 2015-12-10 16:26:02
tags: 
- cache
- computer architecture
categories: computer architecture
toc: false
---

Cache访问失效分为强制性失效/冷失效（compulsory miss/cold miss）、容量失效（capacity miss）和冲突失效（conflict miss）。其中容量失效和冲突失效概念非常相近，理解起来不容易区别。

<!--more-->
**Title: [容量失效（capacity miss）与冲突失效（conflict miss）的区别](https://aidaizyy.github.io/cache_miss_type)**
**Author: [Yunyao Zhang(张云尧)](http://aidaizyy.github.io)**
**E-mail: <aidaizyy@gmail.com>**
**Last Modified: [2015-12-10](http://aidaizyy.github.io)**

# 概念

- **强制性失效：**CPU第一次访问相应cache块，cache中肯定没有该cache块，引起的失效叫做强制性失效。这是不可避免的。
- **容量失效：**有限的cache容量导致cache放不下而替换出cache块，被替换出去的cache块再被访问，引起的失效叫做容量失效。
- **冲突失效：**在直接相联或组相联的cache中，不同的cache块由于index相同相互替换，引起的失效叫做冲突失效。

#理解

如果两个cache块指向同一个cache位置，替换后，访问被替换cache块到底是属于容量失效还是冲突失效呢？
主要看当前cache的存储情况。
假设这里有32KB大小的直接相联cache。
情况一（容量失效）：如果有一个64KB大小的数组需要重复访问，数组的大小远远大于cache大小，没办法全部放入cache。第一次访问数组发生的失效全都是强制性失效。之后再访问数组，再发生的失效则全都是容量失效，这时cache已经存满，容量不足以存储全部数据。
情况二（冲突失效）：如果有两个8KB大小的数据需要来回访问，但是这两个数组都映射到相同的地址，cache大小足够存储全部的数据，但是因为相同地址发生了冲突需要来回替换，发生的失效则全都是冲突失效（第一次访问失效依旧是强制性失效），这时cache并没有存满。
避免容量失效只能通过增加cache大小实现，而避免冲突失效则可以通过提高相联度，优化替换策略，优化代码，增大cache容量等很多措施实现。
