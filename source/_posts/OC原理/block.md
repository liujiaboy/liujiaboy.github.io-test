---
title: block
date: 2021-05-22 16:56:34
tags:
    - Objective-C,
    - iOS
    - block
categories:
    - OC原理
---


# Block的类型

Block分为 Malloc Block、 Stack Block、Global Block，但是怎么做区分呢？


# 循环引用

```

typedef void(^testBlock)(void);
@interface BlockViewController ()

@property (nonatomic, copy) testBlock block;
@property (nonatomic, copy) NSString *name;

@end

@implementation BlockViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // 循环引用
    self.name = @"blcok demo";
    [self blockDemo];
}

- (void)blockDemo {
    self.block = ^(void){
        NSLog(@"%@", self.name);
    };
    self.block();
}

- (void)dealloc {
    NSLog(@"dealloc 来了");
}

@end
```

从A页面进来`BlockViewController`，点击返回，就发现没有走dealloc，因为发生了循环引用。

在BlockVC中，self 持有 block， block持有self，就造成了循环引用。导致无法释放。

self -> block -> self
怎么解决这个问题，按照这个环的形成，正常来说应该有2种解决方案：
## 打破第一个环，self -> block。
    1. 现在是使用的copy修饰的block，如果换成weak修饰是否可以打破？
    2. 答案是不行的，因为使用weak修饰，没有被强持有，初始化之后就会被释放掉，block压根不会执行。
    3. 所以就只能使用第二种方案了。
## 打破第二个环，block -> self。

### weak strong dance
首先我们使用__weak来处理。

```
- (void)blockDemo {
    __weak typeof(self) weakSelf = self;
    self.block = ^(void){
        NSLog(@"%@", weakSelf);
    };
    self.block();
}
```

确实可以解决引用循环。但是会有一个问题，__weak持有的self，是存放在weak表中的，如果，self被释放之后，weakSelf也会被释放掉，整个weak表都会释放。所以当延后执行就会发生问题了。

```
- (void)blockDemo {
    __weak typeof(self) weakSelf = self;
    self.block = ^(void){
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"%@", weakSelf.name);
        });
    };
    self.block();
} 
```

我们延后2s打印数据，进来之后，直接退出，发现打印的就是空。这不符合我们的使用。所以就添加了__strong来修饰weak。

```
- (void)blockDemo {
    __weak typeof(self) weakSelf = self;
    self.block = ^(void) {
        __strong __typeof(weakSelf)strongSelf = weakSelf; // 可以释放 when
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"%@",strongSelf.name);
        });
    };
    self.block();
} 
```

这个时候就没有问题了。

weak-strong-dance是系统自动帮助我们解决引用循环。

### 中介者模式
我们只需要破坏掉block -> self的这个环就可以了，处理weak-strong之外，还有另外一种方法，中介者模式。

中介者模式则需要我们手动解决循环引用。

1. 使用__block 创建一个变量，替代self
    
    ```
    - (void)blockDemo {
        __block ViewController *vc = self;
        self.block = ^(void){
            dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
                NSLog(@"%@",vc.name);
                vc = nil;
            });
        };
        self.block();
    }
    ```
    
    使用__block修饰代替self，然后在block执行完毕时，重新置空，也可以打破block持有self的环。
    
2. 修改block，添加参数
    
    ```
    typedef void(^testBlock)(ViewController *);
    
    - (void)blockDemo {
        self.block = ^(ViewController *vc){
            dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
                NSLog(@"%@",vc.name);
            });
        };
        self.block(self);
    }
    ```

    我们可以不让block持有self，可以把self对应的`BlockViewController`当做参数传到block中，这样也可以打破block持有self的环。
    
当然还有其他的方式可以解决，毕竟怎么传值的方式有很多种，比如NSProxy等。

# Block clang分析

我们在main.m文件中添加一个最简单的block，然后通过clang进行编译，查看main.cpp文件，看看block的一些底层原理。

我们直接创建一个main.m文件。不需要使用工程，当然也可以创建一个工程，在main.m中添加代码：

```
#include "stdio.h"

int main() {

    void (^block)() = ^(){
        printf("hello block");
    };
    block();
    
    return 0;
}
```

接下来，我们使用clang命令编译该文件
`clang -rewrite-objc main.c -o main.cpp`

或者 使用

```
clang -rewrite-objc -fobjc-arc -fobjc-runtime=ios-14.0.0 -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator14.3.sdk main.m
```

我们通过main.cpp文件学习一下block

## 没有外部变量的block

```
int main(){

    void (*block)() = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA));
    ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);

    return 0;
}
```

看起来很乱，我们简化一下：

```
int main(){
    // block = __main_block_impl_0(参数1，参数2)
    void (*block)() = __main_block_impl_0(__main_block_func_0, &__main_block_desc_0_DATA);
    // block执行，调用方法，参数是block本身
    block->FuncPtr(block);

    return 0;
}
```

我们先看一下`__main_block_impl_0`：

```
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```

`__main_block_impl_0`即是一个结构体，又是一个`__main_block_impl_0()`方法。结构体内部还有两个结构体`impl`和`Desc`。

```
struct __block_impl {
  void *isa;        // isa指针
  int Flags;        // flags
  int Reserved;     //
  void *FuncPtr;    // 函数
};

static struct __main_block_desc_0 {
  size_t reserved;      //
  size_t Block_size;    // 
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};
```

* impl.isa指针，指向的是stack block
* impl.Flasg = 0。
* impl.FuncPtr = fp，也就是`__main_block_func_0`。这种属于函数式，函数做为参数。
* Desc = desc，&__main_block_desc_0_DATA

而`__main_block_func_0`这个方法长这样：

```
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
    printf("hello block");
}
```

函数有一个参数就是block本身。之所以需要当成参数传进来，是因为，方便block获取值来使用，在接下来的block引入外部变量一看就知道了。

## block引入外部变量

改变一些main.m，加入一个外部变量a。

```
int a = 10;
void (^block)() = ^(){
    printf("hello block = %d", a);
};
block();
```

重新使用clang编译一下：

```
int main(){

    int a = 10;
    void (*block)() = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, a));
    ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);

    return 0;
}
```

再次简化一下clang编译之后的代码：

```
int a = 10;
void (*block)() = __main_block_impl_0(__main_block_func_0, &__main_block_desc_0_DATA, a);
block->FuncPtr(block);
```

当引入外部变量之后，`__main_block_impl_0()`变成了3个参数，变量a也被传进去了。

```
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  int a; // blcok内部多了一个a的变量
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _a, int flags=0) : a(_a) {    // a进行直接赋值（c++函数）
    impl.isa = &_NSConcreteStackBlock;  // 还是stack block
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
    // a重新生成了一个变量，这个a和外界的变量是同一个值，但是是不同的两块地址，都指向10
    int a = __cself->a; // bound by copy
    printf("hello block = %d", a);
}
```

当引入外部变量是，会在block的结构体中会增加一个与外界同名的变量。
在block函数内部，会重新生成一个变量a来指向block->a

这种操作，相当于 

```
int a = 10; // 相当于block的外部变量
int b = a;  // b相当于block函数__main_block_func_0中生成的a
printf("pa=%p, pb=%p", &a, &b);
```

可以试一下，打印a和b的地址，是两个不同的栈空间，指向同一个值。

`pa=0x7ffee2c23c1c, pb=0x7ffee2c23c18`

这也就是我们没有办法在block内部操作外界变量的原因，在block内没有拿到外界的地址，而是重新生成了一份。

## __block修饰的外部变量

```
__block int a = 10;
void (^block)() = ^(){
    a++;
    printf("hello block = %d", a);
};
block();
```

clang编译之后

```
int main(){
    // 变量a做了处理，变成了__Block_byref_a_0这种结构体
    __Block_byref_a_0 a = {(void*)0,
        (__Block_byref_a_0 *)&a,
        0,
        sizeof(__Block_byref_a_0),
        10};
    // 函数中增加了参数 &a，把地址传过去
    void (*block)() = __main_block_impl_0(__main_block_func_0, &__main_block_desc_0_DATA, &a, 570425344));
    block)->FuncPtr(block);

    return 0;
}
```

我们查看`__Block_byref_a_0`这个结构体，它的初始化方法和传的参数是一一对应的。

```
__Block_byref_a_0 a = {
    (void*)0,                   // __isa，因为是一个常量，所以没有指向
    (__Block_byref_a_0 *)&a,    // __forwarding，指向block外部的变量a的地址
    0,                          // flags
    sizeof(__Block_byref_a_0),  // size
    10                          // block外部变量a的值
};

struct __Block_byref_a_0 {
  void *__isa;
__Block_byref_a_0 *__forwarding;
 int __flags;
 int __size;
 int a;
};
```

我们在看block的结构体`__main_block_impl_0`。多了一个`__Block_byref_a_0`类型的对象。指向block外部生成的对象。

```

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  __Block_byref_a_0 *a; // by ref
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, __Block_byref_a_0 *_a, int flags=0) : a(_a->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
    // 指针指向，
    __Block_byref_a_0 *a = __cself->a; // bound by ref
    // a的__forwarding指向的地址就是外界变量a的地址。
    (a->__forwarding->a)++;
    printf("hello block = %d", (a->__forwarding->a));
}
```

所以这里在block内部执行a++，是可以的。

## block引用对象类型

```
NSMutableArray *mArray = [NSMutableArray array];
Person *p = [Person new];
p.name = @"1";
    
Person *p2 = [Person new];
p2.name = @"2";
[mArray addObject:p2];
    
void (^block)() = ^(){
    NSLog(@"p.name=%@, mArray=%@",p.name, mArray);
};
block();
```

我们声明了两个变量，一个可变数组，一个person对象。clang编译一下：

```
// 可变数组的初始化
NSMutableArray *mArray = objc_msgSend(objc_getClass("NSMutableArray"), sel_registerName("array"));
// person初始化
Person *p = objc_msgSend(objc_getClass("Person"), sel_registerName("new"));
// setName
objc_msgSend(p, sel_registerName("setName:"), (NSString *)&__NSConstantStringImpl__var_folders_v1_79z2l10138z4855091f7nkh40000gn_T_main_4445e3_mi_0);
    
// p2初始化
Person *p2 = objc_msgSend(objc_getClass("Person"), sel_registerName("new"));
objc_msgSend(p2, sel_registerName("setName:"), (NSString *)&__NSConstantStringImpl__var_folders_v1_79z2l10138z4855091f7nkh40000gn_T_main_4445e3_mi_1);
// addObject:
objc_msgSend(mArray, sel_registerName("addObject:"), p2);

void (*block)() = __main_block_impl_0(__main_block_func_0, &__main_block_desc_0_DATA, p, mArray, 570425344));
block->FuncPtr(block);
```

这里与使用`__block`修饰的变量不一样了，对象类型的与正常的初始化没有什么区别。
再看看block的结构体，以及调用的方法：

```
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  Person *__strong p;   // __strong 修饰
  NSMutableArray *__strong mArray;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, Person *__strong _p, NSMutableArray *__strong _mArray, int flags=0) : p(_p), mArray(_mArray) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

// block方法调用
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
    Person *__strong p = __cself->p; // bound by copy
    NSMutableArray *__strong mArray = __cself->mArray; // bound by copy

    NSLog((NSString *)&__NSConstantStringImpl__var_folders_v1_79z2l10138z4855091f7nkh40000gn_T_main_4445e3_mi_2,((NSString *(*)(id, SEL))(void *)objc_msgSend)((id)p, sel_registerName("name")), mArray);
}
```

`__main_block_impl_0`结构体内部，只是增加了两个用strong修饰符修饰的变量。
`__main_block_func_0`方法中，使用两个对象时，重新生成了一个`__strong`修饰的变量。


这里需要注意的是，因为在block内部使用的是`__strong`修饰的变量。在赋值的时候，会进行深拷贝。注意也只是单层深拷贝，内部元素不会做拷贝。
也就是说，外部变量数组mArray，会被拷贝一份放在block内部。但是指向的都是相同的指针。数组中的元素没有变化。

我们添加一些打印信息，然后打印一下：

```
NSMutableArray *mArray = [NSMutableArray array];
Person *p = [Person new];
p.name = @"1";
    
Person *p2 = [Person new];
p2.name = @"2";
[mArray addObject:p2];

NSLog(@"p=%p, p=%@", &p, p);
NSLog(@"mArray=%p, mArray=%p, obj=%@", &mArray, mArray, mArray[0]);
    
void (^block)() = ^(){
    
    NSLog(@"p1=%p, p1=%@", &p, p);
    NSLog(@"mArray1=%p, mArray1=%p, obj1=%@", &mArray, mArray, mArray[0]);
};
block();
```
运行一下，看下结果：

```
p=0x7ffeeb105c10, p=<Person: 0x600000814090>
mArray=0x7ffeeb105c18, mArray=0x600000452a60, obj=<Person: 0x6000008140a0>
-------------- blcok 里
p1=0x60000041fb90, p1=<Person: 0x600000814090>
mArray1=0x60000041fb98, mArray1=0x600000452a60, obj1=<Person: 0x6000008140a0>
```

涉及到[深拷贝和浅拷贝的处理可以看这一篇文章](https://www.jianshu.com/p/df1579149c5c)。


# Block的底层原理

我们用block引入外部`__block`修饰的变量来做例子，看一下：

```
__block NSString *name = [NSString stringWithFormat:@"%@", @"name"];
void(^block)(void) = ^{   // 这一行打断点。运行
    name = @"block";
    NSLog(@"name=%@", name);
};
block();
```

打上断点，打开汇编调试，运行一下：

```
0x1002062b0 <+100>: nop    
0x1002062b4 <+104>: ldr    x10, #0x1d4c              ; (void *)0x0000000253e91a20: _NSConcreteStackBlock
0x1002062b8 <+108>: str    w9, [sp, #0x48]
0x1002062bc <+112>: str    x10, [sp, #0x8]
0x1002062c0 <+116>: nop    
0x1002062c4 <+120>: ldr    d0, 0x100207f68
0x1002062c8 <+124>: adr    x9, #0xa4                 ; __main_block_invoke at main.m:21
```

其实汇编代码后面已经给了注释，我们也读一下这个x10寄存器：

```
(lldb) register read x10
     x10 = 0x0000000253e91a20  libsystem_blocks.dylib`_NSConcreteStackBlock
(lldb) 
```

打印的是一个stack block，我们知道引入了外部变量，会执行block_copy操作，我们再添加一个`_block_copy`的符号断点，继续执行。

进来`_block_copy`之后，发现属于`libsystem_blocks.dylib`这个库，然后我们下载对应的源码。

我们想知道它在内部做了什么。在`_block_copy`的汇编代码中，在最后的return时，加一个断点。在看看寄存器x0的值：

```
(lldb) register read x0
      x0 = 0x00000002818c44e0
(lldb) po 0x00000002818c44e0
<__NSMallocBlock__: 0x2818c44e0>
```

这个block从stack变成了malloc。接下来我们看看源码：

## Block_layout
```
struct Block_layout {
    void *isa;      // isa指针
    volatile int32_t flags; // contains ref count // 标志状态，是一个枚举
    int32_t reserved;   // 保留字段，可能有其他的作用
    BlockInvokeFunction invoke; // 函数执行
    struct Block_descriptor_1 *descriptor; // block的描述信息，size
    // imported variables
};
```

在`_Block_copy`的源码中，第一行就是`Block_layout`，这个才是block真正的样子，一个结构体。

我们在看看flags都有哪些值，表示什么意思：

```
enum {
    BLOCK_DEALLOCATING =      (0x0001),  // runtime 标记正在释放
    BLOCK_REFCOUNT_MASK =     (0xfffe),  // runtime 存储引用计数的值
    BLOCK_NEEDS_FREE =        (1 << 24), // runtime 是否增加或减少引用计数的值
    BLOCK_HAS_COPY_DISPOSE =  (1 << 25), // compiler 是否拥有拷贝辅助函数 确定block是否存在Block_descriptor_2这个参数
    BLOCK_HAS_CTOR =          (1 << 26), // compiler: helpers have C++ code 是否有C++析构函数
    BLOCK_IS_GC =             (1 << 27), // runtime 是否有垃圾回收
    BLOCK_IS_GLOBAL =         (1 << 28), // compiler 是否是全局block
    BLOCK_USE_STRET =         (1 << 29), // compiler: undefined if !BLOCK_HAS_SIGNATURE
    BLOCK_HAS_SIGNATURE  =    (1 << 30), // compiler 是否拥有签名
    BLOCK_HAS_EXTENDED_LAYOUT=(1 << 31)  // compiler 确定Block_descriptor_3中的layout参数
};
```

```
// 这个结构体是block_layout必有的变量
#define BLOCK_DESCRIPTOR_1 1
struct Block_descriptor_1 {
    uintptr_t reserved;
    uintptr_t size;
};

// 可选 这两个是可选变量。
#define BLOCK_DESCRIPTOR_2 1
// 当flag=BLOCK_HAS_COPY_DISPOSE时才会存在
struct Block_descriptor_2 {
    // requires BLOCK_HAS_COPY_DISPOSE
    BlockCopyFunction copy;
    BlockDisposeFunction dispose;
};


#define BLOCK_DESCRIPTOR_3 1
// 当flag=BLOCK_HAS_SIGNATURE时才会存在
struct Block_descriptor_3 {
    // requires BLOCK_HAS_SIGNATURE
    const char *signature;
    const char *layout;     // contents depend on BLOCK_HAS_EXTENDED_LAYOUT
};
```

Block_descriptor_3当中存放的是block的签名信息。我们知道block是一个匿名函数，只要是一个函数就会有签名，比如`v8@?0`这种样式的就是签名。

其中`v`表示返回值是`void`，`@?`表示未知的对象，即为`block`。
这和方法签名是有所不同的，方法签名一般是`v@:`这样的形式(此处只说返回值为void的场景)，`:`表示`SEL`。

接下来，我们看下`_Block_copy`。

## _Block_copy

```
void *_Block_copy(const void *arg) {
    // 创建一个新的block
    struct Block_layout *aBlock;
    // arg就是栈上的block
    if (!arg) return NULL;
    
    // 直接赋值，新创建的aBlock指向arg
    aBlock = (struct Block_layout *)arg;
    // 我们已经对flags的值做了解释，这里做判断，其实这里就是一个堆区的block
    if (aBlock->flags & BLOCK_NEEDS_FREE) {
        // latches on high
        // 处理refcount相关（引用计数）
        latching_incr_int(&aBlock->flags);
        // 直接返回
        return aBlock;
    }
    else if (aBlock->flags & BLOCK_IS_GLOBAL) {
        // 全局区block，直接返回
        return aBlock; // 不需要
    }
    else { 
        // 这里就只有可能是栈区的block
        // Its a stack block.  Make a copy.
        // 申请内存空间，大小与aBlock一样
        struct Block_layout *result =
            (struct Block_layout *)malloc(aBlock->descriptor->size);
        if (!result) return NULL;
        // 将栈区的数据copy到堆区
        memmove(result, aBlock, aBlock->descriptor->size); // bitcopy first
#if __has_feature(ptrauth_calls)
        // Resign the invoke pointer as it uses address authentication.
        // 设置invoke，这样堆上的block调用才会与栈上一致
        result->invoke = aBlock->invoke;
#endif
        // reset refcount
        // 重置refcount
        result->flags &= ~(BLOCK_REFCOUNT_MASK|BLOCK_DEALLOCATING);    // XXX not needed
        // 设置flags
        result->flags |= BLOCK_NEEDS_FREE | 2;  // logical refcount 1
        _Block_call_copy_helper(result, aBlock);
        // Set isa last so memory analysis tools see a fully-initialized object.
        // 设置isa的为malloc block
        result->isa = _NSConcreteMallocBlock;
        return result;
    }
}

// 处理引用计数
static int32_t latching_incr_int(volatile int32_t *where) {
    while (1) {
        int32_t old_value = *where;
        if ((old_value & BLOCK_REFCOUNT_MASK) == BLOCK_REFCOUNT_MASK) {
            return BLOCK_REFCOUNT_MASK;
        }
        if (OSAtomicCompareAndSwapInt(old_value, old_value+2, where)) {
            return old_value+2;
        }
    }
}
```

block copy主要做了以下操作：
* malloc block处理引用计数，直接返回
* global block不做任何处理
* stack block
    1. 申请内存空间
    2. 将栈区的数据拷贝到堆区
    3. 设置isa指向malloc block

## Block_byref

还记得上面的引用`__block`变量的block，clang编译之后的样子吗？改用NSString变量之后，又是另一种风味。

```
int main(int argc, char * argv[]) {
    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 
        __attribute__((__blocks__(byref))) __Block_byref_name_0 name = {
            (void*)0,
            (__Block_byref_name_0 *)&name,
            33554432,
            sizeof(__Block_byref_name_0),
            __Block_byref_id_object_copy_131,
            __Block_byref_id_object_dispose_131,
            ((NSString * _Nonnull (*)(id, SEL, NSString * _Nonnull, ...))(void *)objc_msgSend)((id)objc_getClass("NSString"), sel_registerName("stringWithFormat:"), (NSString *)&__NSConstantStringImpl__var_folders_nw_tqjtztpn1yq6w0_wmgdvn_vc0000gn_T_main_41740c_mi_0, (NSString *)&__NSConstantStringImpl__var_folders_nw_tqjtztpn1yq6w0_wmgdvn_vc0000gn_T_main_41740c_mi_1)
        };
        
        void(*myBlock)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (__Block_byref_name_0 *)&name, 570425344));
        ((void (*)(__block_impl *))((__block_impl *)myBlock)->FuncPtr)((__block_impl *)myBlock);

        return UIApplicationMain(argc, argv, __null, NSStringFromClass(((Class (*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("AppDelegate"), sel_registerName("class"))));
    }
}

struct __Block_byref_name_0 {
    void *__isa;                                        // 8
    __Block_byref_name_0 *__forwarding;                 // 8
    int __flags;                                        // 4
    int __size;                                         // 4
    void (*__Block_byref_id_object_copy)(void*, void*); // 8
    void (*__Block_byref_id_object_dispose)(void*);     // 8
    NSString *name;
};
```

__block修饰的name对象被转化成了一个__Block_byref_name_0的结构体，在源码中也有一个`Block_byref`与之对应。

```
struct Block_byref {
    void *isa;
    struct Block_byref *forwarding;
    volatile int32_t flags; // contains ref count
    uint32_t size;
};

// 可选变量
struct Block_byref_2 {
    // requires BLOCK_BYREF_HAS_COPY_DISPOSE
    BlockByrefKeepFunction byref_keep;
    BlockByrefDestroyFunction byref_destroy;
};

// 可选变量
struct Block_byref_3 {
    // requires BLOCK_BYREF_LAYOUT_EXTENDED
    const char *layout;
};
```

我们把`__Block_byref_name_0`和`Block_byref`放在一起比较一下：

```
__Block_byref_name_0                ->  Block_byref
(void*)0,                           ->  isa
(__Block_byref_name_0 *)&name,      ->  forwarding
33554432,                           ->  flags
sizeof(__Block_byref_name_0),       ->  size
__Block_byref_id_object_copy_131,   ->  byref_kep
__Block_byref_id_object_dispose_131,->  byref_destroy
```

在上面的章节中，有一个点没有说，这里再重新说一下，这个外界的变量是怎么被拷贝进block里头的。

```
void(*myBlock)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (__Block_byref_name_0 *)&name, 570425344));
```

在block声明的时候，还记得吧，其中有`__main_block_desc_0_DATA`这个参数。

```
static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
  void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
  void (*dispose)(struct __main_block_impl_0*);
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0};
```

`__main_block_desc_0_DATA`的生成也是通过方法调用来产生的，而变量的拷贝就是发生在这里`__main_block_copy_0`。

```
static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {
    _Block_object_assign((void*)&dst->name, (void*)src->name, 8/*BLOCK_FIELD_IS_BYREF*/);
}
```

接下来看看`_Block_object_assign`是怎么实现的，在源码里头：

```
//_Block_object_assign有三个参数，第三个参数与下面的枚举值相对应。
enum {
    BLOCK_FIELD_IS_OBJECT  =  3,  // 截获的是对象 __attribute__((NSObject)), block, ...
    BLOCK_FIELD_IS_BLOCK   =  7,  // 截获的是block变量（不是block的参数）
    BLOCK_FIELD_IS_BYREF    =  8,  // 截获的是__block修饰的对象
    BLOCK_FIELD_IS_WEAK     = 16,  // 截获的是__weak修饰的对象
    BLOCK_BYREF_CALLER     = 128, // called from __block (byref) copy/dispose support routines.
};

// 根据传入的对象的类型
void _Block_object_assign(void *destArg, const void *object, const int flags) {
    const void **dest = (const void **)destArg;
    switch (os_assumes(flags & BLOCK_ALL_COPY_DISPOSE_FLAGS)) {
        // 对象类型
        case BLOCK_FIELD_IS_OBJECT:
            // retatin count +1，
            _Block_retain_object(object);
            // 持有object，多了一次强引用
            *dest = object;
            break;
        // block
        case BLOCK_FIELD_IS_BLOCK:
            // 把block从栈上拷贝到堆上，详情看上方_Block_copy的分析，只是参数变化
            *dest = _Block_copy(object);
            break;
        
        case BLOCK_FIELD_IS_BYREF | BLOCK_FIELD_IS_WEAK:
        // __block修饰的变量
        case BLOCK_FIELD_IS_BYREF:
            *dest = _Block_byref_copy(object);
            break;
        
        case BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_OBJECT:
        case BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_BLOCK:
            *dest = object;
            break;

        case BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_OBJECT | BLOCK_FIELD_IS_WEAK:
        case BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_BLOCK  | BLOCK_FIELD_IS_WEAK:
            *dest = object;
            break;
        
        default:
            break;
    }
}
```

### __block修饰的变量拷贝

```
static struct Block_byref *_Block_byref_copy(const void *arg) {
    
    // Block_byref  结构体
    struct Block_byref *src = (struct Block_byref *)arg;

    if ((src->forwarding->flags & BLOCK_REFCOUNT_MASK) == 0) {
        // src points to stack
        // 创建新值，申请内存空间
        struct Block_byref *copy = (struct Block_byref *)malloc(src->size);
        copy->isa = NULL;
        // byref value 4 is logical refcount of 2: one for caller, one for stack
        // 设置flags
        copy->flags = src->flags | BLOCK_BYREF_NEEDS_FREE | 4;
        
        // copy的forwarding指向自己，自己在堆里
        copy->forwarding = copy; // patch heap copy to point to itself
        // src是老值，指向的也是自己。指向的是堆，所以block内部改变，外部也会改变
        src->forwarding = copy;  // patch stack to point to heap copy
        
        copy->size = src->size;

        if (src->flags & BLOCK_BYREF_HAS_COPY_DISPOSE) {
            // Trust copy helper to copy everything of interest
            // If more than one field shows up in a byref block this is wrong XXX
            // 通过内存偏移，获取Block_byref_2
            struct Block_byref_2 *src2 = (struct Block_byref_2 *)(src+1);
            struct Block_byref_2 *copy2 = (struct Block_byref_2 *)(copy+1);
            // 存值
            copy2->byref_keep = src2->byref_keep;
            copy2->byref_destroy = src2->byref_destroy;

            // 这里会判断，有没有Block_byref_3，有的话也是通过内存地址偏移来获取
            if (src->flags & BLOCK_BYREF_LAYOUT_EXTENDED) {
                struct Block_byref_3 *src3 = (struct Block_byref_3 *)(src2+1);
                struct Block_byref_3 *copy3 = (struct Block_byref_3*)(copy2+1);
                copy3->layout = src3->layout;
            }
            // 执行拷贝__Block_byref_id_object_copy
            (*src2->byref_keep)(copy, src);
        }
        else {
            // Bitwise copy.
            // This copy includes Block_byref_3, if any.
            memmove(copy+1, src+1, src->size - sizeof(*src));
        }
    }
    // already copied to heap
    else if ((src->forwarding->flags & BLOCK_BYREF_NEEDS_FREE) == BLOCK_BYREF_NEEDS_FREE) {
        latching_incr_int(&src->forwarding->flags);
    }
    
    return src->forwarding;
}
```

在新生成的结构体`__Block_byref_name_0`中，还有一个名为`__Block_byref_id_object_copy_131`的方法:

```
static void __Block_byref_id_object_copy_131(void *dst, void *src) {
 _Block_object_assign((char*)dst + 40, *(void * *) ((char*)src + 40), 131);
}
```

就是根据地址便宜来获取name的值，这里偏移的是40个字节。看一下`__Block_byref_name_0`中的变量，name上边的所有变量加起来所占的内存是40个字节。所以这里是对name做了一次拷贝。

对于`__block`修饰的变量可以在block内部可以修改，主要是因为：
1. block从栈区，拷贝到堆区
2. __block修饰的变量`name`，会生成一个新的结构体`__Block_byref_name_0`，拷贝到blcok内部
3. 对元类的值进行拷贝，并修改原来值的指向（指向为block内部值）

### block释放
`__main_block_desc_0_DATA`中还有另外一个参数`__main_block_dispose_0`.

```
static void __main_block_dispose_0(struct __main_block_impl_0*src) {
    _Block_object_dispose((void*)src->lg_name, 8/*BLOCK_FIELD_IS_BYREF*/);
}
```

我们看一下`_Block_object_dispose`是怎么释放的。

```
void _Block_object_dispose(const void *object, const int flags) {
    switch (os_assumes(flags & BLOCK_ALL_COPY_DISPOSE_FLAGS)) {
        case BLOCK_FIELD_IS_BYREF | BLOCK_FIELD_IS_WEAK:
        // 如果是__block修饰的，使用_Block_byref_release
        case BLOCK_FIELD_IS_BYREF:
            // get rid of the __block data structure held in a Block
            _Block_byref_release(object);
        break;
        // block
        case BLOCK_FIELD_IS_BLOCK:
            _Block_release(object);
        break;
      // 对象类型的变量
      case BLOCK_FIELD_IS_OBJECT:
            // 调用系统方法，不用处理
            _Block_release_object(object);
        break;
      case BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_OBJECT:
      case BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_BLOCK:
      case BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_OBJECT | BLOCK_FIELD_IS_WEAK:
      case BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_BLOCK  | BLOCK_FIELD_IS_WEAK:
        break;
      default:
        break;
    }
}
```

`_Block_byref_release`的释放，消耗新创建的Block_byref结构体。

```
static void _Block_byref_release(const void *arg) {
    struct Block_byref *byref = (struct Block_byref *)arg;

    // dereference the forwarding pointer since the compiler isn't doing this anymore (ever?)
    byref = byref->forwarding;
    
    if (byref->flags & BLOCK_BYREF_NEEDS_FREE) {
        int32_t refcount = byref->flags & BLOCK_REFCOUNT_MASK;
        os_assert(refcount);
        if (latching_decr_int_should_deallocate(&byref->flags)) {
            if (byref->flags & BLOCK_BYREF_HAS_COPY_DISPOSE) {
                struct Block_byref_2 *byref2 = (struct Block_byref_2 *)(byref+1);
                (*byref2->byref_destroy)(byref);
            }
            free(byref);
        }
    }
}
```

block的释放

```
void _Block_release(const void *arg) {
    struct Block_layout *aBlock = (struct Block_layout *)arg;
    if (!aBlock) return;
    // 全局block不用释放
    if (aBlock->flags & BLOCK_IS_GLOBAL) return;
    // 还有引用计数则没办法释放
    if (! (aBlock->flags & BLOCK_NEEDS_FREE)) return;
    
    if (latching_decr_int_should_deallocate(&aBlock->flags)) {
        _Block_call_dispose_helper(aBlock);
        _Block_destructInstance(aBlock);
        free(aBlock);
    }
}
```

# 总结

* block的分类：global block、stack block，malloc block
* block的的内部实现
* block调用外部变量
* __block修饰的变量
