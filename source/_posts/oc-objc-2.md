---
title: 2.内存对齐
date: 2021-04-17 18:34:07
tags:
    - Objective-C,
    - runtime
    - iOS
categories:
    - Objective-C
---

# 1. 对象的内存对齐

先看代码哈~

```
@interface Person : NSObject

@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) int age;
@property (nonatomic, assign) int score;
@property (nonatomic, assign) long height;
@property (nonatomic) char c1;
@property (nonatomic) char c2;

@end

main() {

Person *person = [Person alloc];
person.name = @"name";
person.age = 18;
person.score = 20;
person.height = 180;
person.c1 = 'a';
person.c2 = 'b';

NSLog(@"%@ - %lu - %lu - %lu",person,sizeof(person),class_getInstanceSize([Person class]),malloc_size((__bridge const void *)(person)));
        
}
```

我们在NSLog上打个断点。有时间的话，可以把属性先注释掉，从0开始，一个属性一个属性的加起来看输出的是什么结果。

1. po 这里直接输出person指向的对象的内存地址。

    ```
    (lldb) po person
    <Person: 0x600002adbcf0>
    ```

2. 这里有一个需要介绍的点，上一章中有用到x person命令，输出的内容与`View Memory`中显示的是一致的。而`x/8gx`就是进行排序。8代表的是输出8组内存。如果是4那就是输出4组内容。每一块都是8个字节。

    ```
    (lldb) x/8gx person
    0x600002adbcf0: 0x0000000103930808 0x0000001200006261
    0x600002adbd00: 0x0000000000000014 0x000000010392b038
    0x600002adbd10: 0x00000000000000b4 0x0000000000000000
    0x600002adbd20: 0x0000c1c5c19bbd20 0x00000000000007fb
    ```
    0x600002adbcf0：是person指向的首地址。后面存放的都是属性的值。内存都是连续的。
    
3. 我们分别输出内存里的内容。
    ```
    (lldb) po 0x0000000103930808
    Person  // isa指针
    
    (lldb) po 0x00000012
    18
    
    (lldb) po 61
    61
    
    (lldb) po 0x0000000000000014
    20
    
    (lldb) po 0x000000010392b038
    name
    
    (lldb) po 0x00000000000000b4
    180
    ```
    
    0x0000001200006261：这一块地址上内容是被拆开的。我们知道int是4个字节，char是1个字节，所以前面的几位是int的值，后面的再进行拆分，分别是两个char类型的数据。
    
    苹果在内存上也是做了足够多的优化，虽然在内存上是16个字节对齐的，但是在内部实现上，还是进行了大量的优化，但是一定要注意的是，属性是按8个字节对齐的。
    
    但是为啥两个int类型的数据没有放在一起呢？可能是系统内部做的优化，可以试一下，把所有的char类型注释掉，两个int类型的数据就会存放在一起。可能是会将char类型的数据优先进行填充吧。另外可以多试一下，3个char类型的会怎么样？
    
4. 最后输出的结果是
    ```
    <Person: 0x600002adbcf0> - 8 - 40 - 48
    ```
    我们分析一下输出的内容：
    
    * person：当前的类对象，存放的指针，指向的内存地址。
    * sizeof(person)：person存放的就是一个指针，8个字节。
    * class_getInstanceSize([Person class])：这个类真正需要的空间。属性是8个字节对齐的。
    * malloc_size((__bridge const void *)(person))：内存中需要开辟的空间。内存空间是16个字节对齐的。

# 1.1 float、double
加如改变其中的一个属性位double类型的，又会是什么情况呢？我这里把上面的height改为了double类型。看一下输出结果

```
(lldb) x/8gx person
0x600000045530: 0x000000010626d808 0x0000001200006261
0x600000045540: 0x0000000000000014 0x0000000106268038
0x600000045550: 0x4066800000000000 0x0000000000000000
0x600000045560: 0x0000000000000000 0x0000000000000000
(lldb) po 0x4066800000000000
4640537203540230144
```

诶~~~ 怎么没有输出180呢？是因为对于float、double类型的数据，系统会做一次特殊的转换处理。我们没有办法直接从内存中读出double类型的值。（也有可能是我没有找到方法，有知道的小伙伴请告知，谢谢！）

但是我们可以通过转化double类型的数据来看是否位上面对应的值。

```
(lldb) p/x (double)180.0
(double) $4 = 0x4066800000000000
```
转换后，发现正好是对应的数据。


# 2. 结构体的内存对齐

>1. 数据成员对⻬规则：结构体(struct)的数据成员，第一个数据成员放在offset为0的地方，以后每个数据成员存储的起始位置要从该成员大小或者成员的子成员大小(只要该成员有子成员，比如说是数组，结构体等)的整数倍开始(比如int为4字节,则要从4的整数倍地址开始存储。

上代码，看看结构体的内存：
```
struct Struct1 {
    double a;   
    char b;     
    int c;      
    short d;    
}str1;

struct Struct2 {
    double a; 
    int b;    
    char c;   
    short d;  
}str2;

NSLog(@"str1 = %lu, str2-%lu",sizeof(str1),sizeof(str2));
```

输出的结果很明白的哈。` str1 = 24, str2-16`

我们分析一下：

```
struct Struct1 {
    double a;   // 8 (0-7)
    char b;     // 1 [8 1] (8)
    int c;      // 4 [9 10 11 12] 9不是4的整数倍(12 13 14 15)
    short d;    // 2 [16 17]
}str1;

// 内部需要的大小为: 17
// 最大属性 : 8
// 结构体整数倍: 24

struct Struct2 {
    double a;   //8 (0-7)
    int b;      //4 (8 9 10 11)
    char c;     //1 (12)
    short d;    //2 (13 14) 13不是2的整数倍，从14开始(14 15)
}str2;

// 内部需要的大小为: 15
// 最大属性 : 8
// 结构体整数倍: 16
NSLog(@"str1 = %lu, str2-%lu",sizeof(str1),sizeof(str2));
```

这就是内存补齐的内容

# 3. 总结：
* 类对象的属性是8个字节对齐的，但是在内存空间是16个字节对齐。
* x/4gx的使用
* 结构体的对齐规则。
    
    
    




