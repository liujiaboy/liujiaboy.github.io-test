---
title: ARM汇编-5 OC反汇编
date: 2021-04-28 14:30:30
tags:
    - ARM汇编
categories:
    - 逆向
---

OC代码的精髓其实就是`objc_msgSend`。而OC的反汇编其实就是查看其中的方法调用。

objc_msgSend有两个参数，第一个是id类型，第二个是SEL类型。id、SEL其实都是一个结构体，内部有isa指针，所以这两个在内存中占有8个字节。


# 1.

声明一个Person类，并添加两个属性，一个类方法。

```
@interface Person : NSObject
@property(nonatomic, copy) NSString * name;
@property(nonatomic, assign) int age;

+(instancetype)person;

@end

@implementation Person

+ (instancetype)person {
    return [[self alloc] init];
}

@end

int main(int argc, char * argv[]) {
    Person * p = [Person person];
    return 0;
}
```

放在main函数里，直接调用类方法，生成一个临时变量。
打上断点，直接在汇编模式下进行debug。

```
Demo`main:
    ...
    ...
    0x10405a16c <+24>:  adrp   x8, 3
    0x10405a170 <+28>:  add    x8, x8, #0x648            ; =0x648 
->  0x10405a174 <+32>:  ldr    x0, [x8]

    0x10405a178 <+36>:  adrp   x8, 3
    0x10405a17c <+40>:  add    x8, x8, #0x638            ; =0x638 
    0x10405a180 <+44>:  ldr    x1, [x8]
    0x10405a184 <+48>:  bl     0x10405a4d4               ; symbol stub for: objc_msgSend
    0x10405a188 <+52>:  mov    x29, x29
    0x10405a18c <+56>:  bl     0x10405a4f8               ; symbol stub for: objc_retainAutoreleasedReturnValue
    0x10405a190 <+60>:  add    x8, sp, #0x8              ; =0x8 
    0x10405a194 <+64>:  str    x0, [sp, #0x8]
    0x10405a198 <+68>:  stur   wzr, [x29, #-0x4]
    0x10405a19c <+72>:  mov    x0, x8
    0x10405a1a0 <+76>:  mov    x8, #0x0
    0x10405a1a4 <+80>:  mov    x1, x8
    0x10405a1a8 <+84>:  bl     0x10405a510               ; symbol stub for: objc_storeStrong
    ...
    ...  
```

这里隐藏了开辟栈空间和回收相关的代码。

`objc_msgSend`需要两个参数id和SEL，从上面的代码可以初步判断两个参数的值分别在x0、x1寄存器中。

首先我们看一下x0寄存器中的数据。按照老方法，adrp计算x8的地址是`0x010405d648`。x0的值存放在`0x010405d648`所指向的内存中。

```
(lldb) po 0x010405d648
<Person: 0x10405d648>
(lldb) x 0x010405d648
0x10405d648: 30 d7 05 04 01 00 00 00 68 d6 05 04 01 00 00 00  0.......h.......
0x10405d658: 08 00 00 00 08 00 00 00 10 00 00 00 08 00 00 00  ................

(lldb) po 0x010405d730
Person
```
我们确定了第一个参数是Person类。在看第二个参数：

```
(lldb) x 0x10405d638
0x10405d638: 05 3d 42 8f 01 00 00 00 00 00 00 00 00 00 00 00  .=B.............
0x10405d648: 30 d7 05 04 01 00 00 00 68 d6 05 04 01 00 00 00  0.......h.......
(lldb) po (SEL)0x018f423d05
"person"

```

没有毛病，就是一个方法`person`。1‘05


不同的系统版本，实现的汇编是不一样，iOS11下，汇编对objc_alloc进行了优化，但是没有对init处理。

iOS14 ：没有消息发送，直接objc_alloc_init
iOS11 ： 一次消息发送，objc_alloc, objc_msgSend(self, init)
iOS9  ： 两次消息发送，objc_msgSend(alloc),objc_msgSend(self, init)

