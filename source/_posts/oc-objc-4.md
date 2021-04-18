---
title: 4.类的本质
date: 2021-04-18 19:40:55
tags:
    - Objective-C,
    - runtime
    - iOS
categories:
    - Objective-C
---

# 类

![](isa_metaclass.png)

* 每个实例对象的isa指针指向与之对应的类对象(Class)。
* 每个类对象(Class)都有一个isa指针指向一个唯一的元类(Meta Class)。
* 每一个元类(Meta Class)的isa指针都指向最上层的元类(Meta Class)（图中的NSObject的Meta Class）。最上层的元类(Meta Class)的isa指针指向自己，形成一个回路。
* 每一个元类(Meta Class)的Super Class指向它原本Class的Super Class的Meta Class。最上层的Meta Class的Super Class指向NSObject Class本身。
* 最上层的NSObject Class的Super Class指向nil。
* 只有Class才有继承关系，实例对象与实例对象不存在继承关系。
* 每一个类对象(Class)在内存中都只有一份。