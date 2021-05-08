---
title: KVC原理
date: 2021-05-08 08:39:49
tags:
    - Objective-C,
    - iOS
categories:
    - OC原理
---

# KVC

官方文档：[About Key-Value Coding](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/index.html#//apple_ref/doc/uid/10000107-SW1)

> Key-value coding is a mechanism enabled by the NSKeyValueCoding informal protocol that objects adopt to provide indirect access to their properties. When an object is key-value coding compliant, its properties are addressable via string parameters through a concise, uniform messaging interface. This indirect access mechanism supplements the direct access afforded by instance variables and their associated accessor methods.

键值编码是由`NSKeyValueCoding`非正式协议启用的一种机制，对象采用这种机制来提供对其属性的间接访问。当对象是键值编码兼容的对象时，可以通过简洁，统一的消息传递接口通过字符串参数来访问其属性。这种间接访问机制补充了实例变量及其关联的访问器方法提供的直接访问。

# KVC - API

## 1. 常用方法

* 获取key对应的value：

    ```
    - (nullable id)valueForKey:(NSString *)key;
    ```

* 通过key来设置value：

    ```
    - (void)setValue:(nullable id)value forKey:(NSString *)key;
    ```

* 通过路径取值，一般情况下是model1中有一个model2，获取model2的属性值。

    ```
    - (nullable id)valueForKeyPath:(NSString *)keyPath;
    ```

* 获取对应路径的值，一般情况下是model1中有一个model2，设置model2的属性值。

    ```
    - (void)setValue:(nullable id)value forKeyPath:(NSString *)keyPath;
    ```

* 获取一个可变类型：

    ```
    - (NSMutableArray *)mutableArrayValueForKey:(NSString *)key;
    
    - (NSMutableSet *)mutableSetValueForKey:(NSString *)key;
    ```

* 默认返回YES，如果当前没有设置key对应的属性（没有找到set<key>方法），会按照_key, _iskey, key, iskey的顺序搜索变量。如果返回NO，则不查询。

    ```
    + (BOOL)accessInstanceVariablesDirectly;
    ```

* 如果你在SetValue方法时面给Value传nil，则会调用这个方法

    ```
    - (void)setNilValueForKey:(NSString *)key;
    ```
    
* 如果Key不存在，且KVC无法搜索到任何和Key有关的字段或者属性，则会调用这个方法，默认是抛出异常。

    ```
    - (nullable id)valueForUndefinedKey:(NSString *)key;
    
    - (void)setValue:(nullable id)value forUndefinedKey:(NSString *)key;
    ```

* KVC提供属性值正确性验证的API，它可以用来检查set的值是否正确、为不正确的值做一个替换值或者拒绝设置新值并返回错误原因。

    ```
    - (BOOL)validateValue:(inout id __nullable * __nonnull)ioValue forKey:(NSString *)inKey error:(out NSError **)outError;
    ```


## 2. set、get流程

声明一个Person类，声明4个变量。注意这里没有添加属性（添加属性默认会生成set、get方法），就是为了验证set、get流程。

```
@interface Person : NSObject
{
    NSString *_name;    // 1.
    NSString *_isName;  // 2.
    NSString *name;     // 3.
    NSString *isName;   // 4.
}

@end
```

调用`setValue:forKey`方法，然后打印

```
Person *person = [[Person alloc] init];
    
// KVC - 设置值的过程 setValue 分析调用过程
[person setValue:@"kvc" forKey:@"name"];

// 1.
NSLog(@"%@-%@-%@-%@",person->_name,person->_isName,person->name,person->isName);
// 2.
//NSLog(@"%@-%@-%@",person->_isName,person->name,person->isName);
// 3.
//NSLog(@"%@-%@",person->name,person->isName);
// 4.
//NSLog(@"%@",person->isName);
```     

分别按照顺序1-2-3-4，每次注释一个变量，每次只执行一句`NSLog`，看看打印结果：

```
// 1.
kvc-(nill)-(nill)-(nill)
// 2.
kvc-(nill)-(nill)
// 3.
kvc-(nill)
// 4.
kvc

```

为了进一步验证，可以在`Person.m`中，实现对应的set和get方法，分别打断点，按照1-2-3-4的顺序分别注释，可以进一步验证，set、get的流程。

`setValue:forKey:`：按照`set<key>, _set<key>, setIs<key>`进行设置。有一个执行，其他的不执行。

> 注意：` _setIs<key>`这个方法不会执行。

`valueForKey`：按照`get<key>, <key>, is<key>, _<key>`顺序进行查找。有一个执行，不执行其他的。

这里有官方的设置key-value的流程：[Accessor Search Patterns](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/SearchImplementation.html#//apple_ref/doc/uid/20000955-CJBBBFFA)，写的很详细。

## 3. 流程总结

# 集合类型





