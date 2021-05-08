---
title: KVO原理
date: 2021-05-08 08:39:59
tags:
    - Objective-C,
    - runtime
    - iOS
categories:
    - OC底层原理
---


# FB - 
添加监听时，创建了一个info，在移除的时候，也是创建了一个临时变量info，这个时候通过NSMutableSet获取`member`来获取添加的observer，是怎么获取到的。

重写了`hash`方法和`isEqual:`方法。可以在member的官方解释中查看。