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

### 2.1 Get 流程

1. 按照访问方法`-get<Key>，-<key>，-is<Key>，-_<key>`的顺序进行查找，如果找到执行步奏【5】。否则执行步奏【2】。
2. 在实例中搜索①`countOf<Key>`，②`objectIn<Key>AtIndex:`（与`NSArray`类定义的原始方法`<key>AtIndexes:`相对应）或③`objectsAtIndexes:`（与NSArray方法相对应）。如果找到①，再找到②或③中的一个，则创建一个响应所有NSArray方法的集合代理对象并将其返回。否则，请继续执行步骤【3】。
3. 如果找到了①`countOf<Key>`，没有找到②或③，那么会去找`enumeratorOf<Key>`和`memberOf<Key>:`（对应NSSet类）。如果找到了所有三个方法，则创建一个响应所有NSSet方法的集合代理对象并将其返回。否则，请继续执行步骤【4】。
4. 如果接收器的类方法`+(BOOL)accessInstanceVariablesDirectly`返回YES，则按照`_<key>，_is<Key>，<key>，is<Key>`的顺序搜索实例变量。如果找到，直接获取实例变量的值，然后继续执行步骤【5】。否则，继续执行步骤【6】。
5. 如果获取到的变量是对象指针，则只需返回结果。
    如果该值是可以转换位`NSNumber`类型，则将其存储在NSNumber实例中并返回该实例。
    如果结果是`NSNumber`不支持的类型，请转换为`NSValue`对象并返回该对象。
6. 如果其他所有方法均失败，则调用`valueForUndefinedKey:`。默认情况下会引发异常。

### 2.2 Set流程

1. 按此顺序查找第一个名为`set<Key>:, _set<Key>:, setIsName:`的set方法。如果找到，调用它并完成。
2. 如果没有找到，如果类方法`+(BOOL)accessInstanceVariablesDirectly`返回YES，则按照顺序`_<key>，_is<Key>，<key>，is<Key>`查找实例变量。如果找到，则直接对变量进行赋值。
3. 如果步奏【1】和【2】都失败了，则调用`setValue:forUndefinedKey:`。默认情况下会引发异常。


# 集合类型

## 1. 集合类型
在`Person`类中，声明一个不可变数组

```
@property (nonatomic, copy) NSArray *array;
```

对不可变类型进行赋值时，可以使用`mutableArrayValueForKey`先获取一个可变数组，然后直接赋值就好。
```
person.array = @[@"1",@"2",@"3"];
// 修改数组
// person.array[0] = @"100";// 这种方式不可用
// 1. 获取一个新的数组 - KVC 赋值
NSArray *array = [person valueForKey:@"array"];
array = @[@"100",@"2",@"3"];
[person setValue:array forKey:@"array"];
NSLog(@"%@",[person valueForKey:@"array"]);

// 2. 使用mutableArrayValueForKey
NSMutableArray *mArray = [person mutableArrayValueForKey:@"array"];
mArray[0] = @"200";
NSLog(@"%@",[person valueForKey:@"array"]);
```

输出结果：

```
2021-05-09 10:17:58.004460+0800 KVCDemo[70852:4744247] (
    100,
    2,
    3
)
2021-05-09 10:17:58.005523+0800 KVCDemo[70852:4744247] (
    200,
    2,
    3
)
```

如果声明的是一个可变数组，那通过`[person valueForKey:@"mArray"];`获取到的就是一个可变数组。

## 2. 集合类型set、get流程补充

直接上代码：这里使用的key是一个没有在类中声明的变量/属性`pens`。

```
person.arr = @[@"pen0", @"pen1", @"pen2", @"pen3"];
// 直接运行，在这里会发生crash
NSArray *array = [person valueForKey:@"pens"];
NSLog(@"%@",[array objectAtIndex:1]);
NSLog(@"%d",[array containsObject:@"pen1"]);
    
// set 集合
person.set = [NSSet setWithArray:person.arr];
NSSet *set = [person valueForKey:@"books"];
[set enumerateObjectsUsingBlock:^(id  _Nonnull obj, BOOL * _Nonnull stop) {
    NSLog(@"set遍历 %@",obj);
}];
```

直接运行，会发生crash。

> *** Terminating app due to uncaught exception 'NSUnknownKeyException', reason: '[<LGPerson 0x6000024610c0> valueForUndefinedKey:]: this class is not key value coding-compliant for the key pens.'

按照上面的流程分析，我们需要对NSArray和NSSet类型提供方法。

```
//MARK: - NSArray
// 个数
- (NSUInteger)countOfPens{
    NSLog(@"%s",__func__);
    return [self.arr count];
}

// 获取值
- (id)objectInPensAtIndex:(NSUInteger)index {
    NSLog(@"%s",__func__);
    return [NSString stringWithFormat:@"pens %lu", index];
}

//MARK: - set
// 个数
- (NSUInteger)countOfBooks{
    NSLog(@"%s",__func__);
    return [self.set count];
}

// 是否包含这个成员对象
- (id)memberOfBooks:(id)object {
    NSLog(@"%s",__func__);
    return [self.set containsObject:object] ? object : nil;
}

// 迭代器
- (id)enumeratorOfBooks {
    // objectEnumerator
    NSLog(@"%s",__func__);
    return [self.arr reverseObjectEnumerator];
}
```

补充完整上述方法，即可正常运行，对数组进行操作。

# 结构体

声明一个结构体类型的属性。

```
typedef struct {
    float x, y, z;
} ThreeFloats;

@property (nonatomic) ThreeFloats threeFloats;
```

```
ThreeFloats floats = {1.,2.,3.};
NSValue *value     = [NSValue valueWithBytes:&floats objCType:@encode(ThreeFloats)];
[person setValue:value forKey:@"threeFloats"];
NSValue *value1    = [person valueForKey:@"threeFloats"];
NSLog(@"%@",value1);
    
ThreeFloats th;
[value1 getValue:&th];
NSLog(@"%f-%f-%f",th.x,th.y,th.z);
```

对于结构体类型的数据，需要先转化成`NSValue`类型。常量类型会先转化成`NSNumber`类型


# 自定义KVC

根据set、get流程分析，自定义主要分为以下几个流程，需要注意的是要做安全判断，防止发生异常。

## 1. kvc自定义set

1. 判断key，value的情况
2. 通过传进来的key生成对应的set方法。
3. 判断生成的3种set方法是否可以被响应，可以被响应直接return。
4. 判断accessInstanceVariablesDirectly是否返回YES。
5. 判断4种实例变量是否存在，存在则赋值，否则crash异常处理。

## 2. KVC 自定义Get

1. 判断key的值。
2. 生成对应的`-get<Key>，-<key>，-is<Key>，-_<key>`方法。判断是否可以响应。
3. 不响应判断get流程种NSArray的处理。
4. 不想要判断get流程种NSSet的处理。
5. 判断accessInstanceVariablesDirectly是否返回YES。
6. 判断变量是否存在，存在直接返回。
7. 异常处理。

自定义set、get完全是按照set和get的流程处理的。代码就不上了，太占地方。

# 补充，KVC的高级使用

```
NSArray *arrStr = @[@"1", @"10", @"100"];
NSArray *arrCapStr = [arrStr valueForKey:@"capitalizedString"];
    
for (NSString *str in arrCapStr) {
    NSLog(@"%@", str);
}
    
NSArray *arrCapStrLength = [arrCapStr valueForKeyPath:@"capitalizedString.length"];
for (NSNumber *length in arrCapStrLength) {
    NSLog(@"%ld", (long)length.integerValue);
}
```
    
打印出来的结果：

```
1
10
100
1
2
3
```

还有关于model中嵌套model的也差不多类似，大家探索一下吧。

# 总结

1. KVC可以间接访问私有变量。
2. `valueForKey`返回key对应的类型数据。如果是不可变数组，通过`mutableArrayValueForKey`获取的也会是可变类型。
3. `setValue:forKey:, valueForKey:`的流程。
4. 自定义KVC。
