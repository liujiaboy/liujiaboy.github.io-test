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


# 1. OC汇编

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

没有毛病，就是一个方法`person`。


不同的系统版本，实现的汇编是不一样，iOS11下，汇编对objc_alloc进行了优化，但是没有对init处理。

iOS14 ：没有消息发送，直接objc_alloc_init
iOS11 ： 一次消息发送，objc_alloc, objc_msgSend(self, init)
iOS9  ： 两次消息发送，objc_msgSend(alloc),objc_msgSend(self, init)

## 1.1 objc_storeStrong
这里还看到一个这个东西，这个设计到oc源码的逻辑，我们稍微看一下。

```
void
objc_storeStrong(id *location, id obj)
{
    id prev = *location;
    if (obj == prev) {
        return;
    }
    objc_retain(obj);
    *location = obj;
    objc_release(prev);
}
```

这个函数需要两个参数，第一个是id *类型，这就是一个地址，第二个是id类型。我们反过来看汇编代码，看这两个变量，正常来说还是在x0、x1寄存器中。

```
// x8中存地址
0x10405a190 <+60>:  add    x8, sp, #0x8              ; =0x8 
// 把x0寄存器的值放在x8中。
0x10405a194 <+64>:  str    x0, [sp, #0x8]
// 把0存起来
0x10405a198 <+68>:  stur   wzr, [x29, #-0x4]
// x0 = x8，是一个地址。
0x10405a19c <+72>:  mov    x0, x8
// x8置空
0x10405a1a0 <+76>:  mov    x8, #0x0
// x1 = 0
0x10405a1a4 <+80>:  mov    x1, x8
// 这里x0是一个地址， x1是个nil，两个变量
0x10405a1a8 <+84>:  bl     0x10405a510  ; symbol stub for: objc_storeStrong
```

通过分析汇编，objc_storeStrong两个变量分别是一个地址，一个是nil。然后看一些源码。

```
void
objc_storeStrong(id *location, id obj)
{
    // location有值， obj = nil
    id prev = *location;
    // 不相等
    if (obj == prev) {
        return;
    }
    // 对nil进行retain
    objc_retain(obj);
    // 寻址之后置空，也就是把id对象置空
    *location = obj;
    // 释放
    objc_release(prev);
}
```
所以这个函数不仅仅只是用来强引用的，还可以进行释放操作，在这里就是一个很明显的例子。

# 1.2 属性

我们在mian函数中，对实例对象p的两个属性进行赋值。
```
int main(int argc, char * argv[]) {
    Person * p = [Person person];
    p.name = @"name";
    p.age = 18;
    
    return 0;
}
```

在真机上执行一下，然后我们使用之前的Hopper工具进行看一下。

![](/images/arm-5-property.jpg)

这里就很详细的标注了整个内容，看起来比读汇编代码省事很多。

# 3. block的汇编

在main函数中直接声明一个栈区的block，全局区的也是一样的道理。

```
int a = 10;
void(^block)(void) = ^() {
    NSLog(@"block--%d",a);
};
    
block();
```

然后真机上运行。这里省去了很大一部分的代码，只拿了关键部分的逻辑。

```
Demo`main:
    0x10052a0cc <+36>:  adrp   x10, 2
    0x10052a0d0 <+40>:  ldr    x10, [x10]
->  0x10052a0d4 <+44>:  str    x10, [sp, #0x8]
```

先获取x10寄存器的值，也就是`0x10052c000`,lldb调试一下：

```
(lldb) x 0x10052c000
0x10052c000: 20 9a 44 29 02 00 00 00 0c c8 66 ef 01 00 00 00   .D)......f.....
0x10052c010: 18 a5 52 00 01 00 00 00 24 a5 52 00 01 00 00 00  ..R.....$.R.....
// 这是一个栈block
(lldb) po 0x10052c000
<__NSStackBlock__: 0x10052c000>

// 这里拿到的是0x10052c010地址指向的内存区域，这个就是block的invoke。
(lldb) dis -s 0x010052a518
    0x10052a518: ldr    w16, 0x10052a520
    0x10052a51c: b      0x10052a500
    0x10052a520: udf    #0x0
    0x10052a524: ldr    w16, 0x10052a52c
    0x10052a528: b      0x10052a500
    0x10052a52c: udf    #0xd
    0x10052a530: ldr    w16, 0x10052a538
    0x10052a534: b      0x10052a500
```

这里看一下block的源码：

```
struct Block_layout {
    void *isa;      // 8个字节
    volatile int32_t flags; // contains ref count   //4个字节
    int32_t reserved;   // 4个字节
    BlockInvokeFunction invoke;
    struct Block_descriptor_1 *descriptor;
    // imported variables
};
```

block也是一个结构体，invoke所在的位置，就是isa之后的16个字节。所以我在内存中取的invoke的实现就是偏移了0x10。

接下来，我们用hopper看一下:

![](/images/arm-5-block.jpg)

![](/images/arm-5-block-invoke.jpg)
