---
title: KVO原理
date: 2021-05-08 08:39:59
tags:
    - Objective-C,
    - iOS
categories:
    - OC原理
---

# KVO
官方文档：[Key-Value Observing](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html#//apple_ref/doc/uid/10000177i)

> <b>Important:</b> In order to understand key-value observing, you must first understand key-value coding.

在官方的文档中，有这么一句话，要理解KVO，必须先知道KVC。




# FB - 
添加监听时，创建了一个info，在移除的时候，也是创建了一个临时变量info，这个时候通过NSMutableSet获取`member`来获取添加的observer，是怎么获取到的。

重写了`hash`方法和`isEqual:`方法。可以在member的官方解释中查看。

