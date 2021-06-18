---
title: 4.类的本质
date: 2021-04-18 19:40:55
tags:
    - Objective-C,
    - runtime
    - iOS
categories:
    - OC原理
---

# 1. 对象、类对象、元类




## 1.1 isa指向、superClass指向


```
@interface Person : NSObject

@end

@interface Teacher : Person

@end

// 然后执行
Person *p = [[Person alloc] init];
Teacher *t = [[Teacher alloc] init];

```

我们根据上诉代码进行分析，isa指针的指向。

> 注意，这里是用的是模拟器，Teacher类继承自Person

## 1.2 实例对象的isa
首先我们先看Person的实例p的isa指向情况

```
// 首先打印一下p的内存情况
(lldb) po p
<Person: 0x1006460b0>
// 输出p指针的情况
(lldb) x/4gx p
0x1006460b0: 0x011d8001000083f9 0x0000000000000000
0x1006460c0: 0x0000000000000000 0x0000000000000000

(lldb) p p 0x011d8001000083f9
(long) $6 = 1791002988741

```

这里拿到p指针指向的内存情况，我们知道第一块内存区域存放的是isa指针，直接打印的话，发现就是一串数字，啥也看不出来。还记得上一章中object_getClass反向验证isa指向最后的"&"运算吗？`0x011d8001000083f9`这个值就是isa->bits，我们用它与`ISA_MASK`进行&运算。因为这里是用的真机，所以`ISA_MASK = 0x00007ffffffffff8`，如果是用Mac或者模拟器，根据芯片类型判断是否是ARM64架构还是x86，然后使用对应的值进行换算。

```
// p/x输出内存的16进制
(lldb) p/x 0x011d8001000083f9 & 0x00007ffffffffff8
(long) $2 = 0x00000001000083f8

// 这里po就是Person类
(lldb) po 0x00000001000083f8
Person
```

打印出来是Person。所以isa指向的就是Person类。那我们做一下验证，直接通过`object_getClass`方法来找一下Person这个类。

```
(lldb) p/x object_getClass(p)
(Class) $12 = 0x00000001000083f8 Person
```

是不是发现，Person类的内存地址是一样的。如果再实例化一个p1，看p1->isa指向的和p->isa指向的是否是同一个Person类的内存地址。

```
// 直接使用object_getClass获取类对象。
lldb) p/x object_getClass([LGPerson alloc])
(Class) $26 = 0x00000001000083f8 LGPerson
```
答案是肯定的，Person类在内存中只有一份，也就是说所有的类对象在内存中都只有一份。

## 1.3 类对象的isa
接下来，我们继续寻找Person类对象的isa指向情况。

```
(lldb) p 0x000001a10018d0c5
(long) $6 = 1791002988741

(lldb) x/4gx 0x00000001000083f8
0x1000083f8: 0x00000001000083d0 0x000000010036a140
0x100008408: 0x0000000100645d60 0x0001803000000003

(lldb) p/x 0x00000001000083d0 & 0x00007ffffffffff8
(long) $4 = 0x00000001000083d0
(lldb) po 0x00000001000083d0
Person
```
发现Person类对象的isa指向的还是Person，但是这个Person所在的内存地址与Person类对象不一样。

这里就出现了元类的概念（Meta Class）。

## 1.4 元类的isa

我们继续寻找元类的isa

```
// 0x00000001000083d0是Person元类所在的内存
(lldb) x/4gx 0x00000001000083d0
0x1000083d0: 0x000000010036a0f0 0x000000010036a0f0
0x1000083e0: 0x0000000100714d10 0x0002e03100000003

(lldb) p/x 0x000000010036a0f0 & 0x00007ffffffffff8
(long) $9 = 0x000000010036a0f0
(lldb) po 0x000000010036a0f0
NSObject

// 拿到NSObject的地址继续x/4gx
(lldb) x/4gx 0x000000010036a0f0
0x10036a0f0: 0x000000010036a0f0 0x000000010036a140
0x10036a100: 0x00000001007877b0 0x0003e03100000007
```

使用相同的方法找到元类的`isa`指向的是`NSObject`，这个`NSObject`是类对象吗？

对`NSObject`继续`x/4gx`发现`isa`锁指向的内存地址是一样的。

我们通过`object_getClass([[NSObject alloc] init])`来看看`NSObject`类对象的内存

```
// 获取NSObject类的地址，与p/x NSObject.class效果一致
(lldb) p/x object_getClass([NSObject alloc])
(Class) $13 = 0x000000010036a140 NSObject

(lldb) x/4gx 0x000000010036a140
0x10036a140: 0x000000010036a0f0 0x0000000000000000
0x10036a150: 0x0000000100786740 0x0002801000000003

// 从这里开始，就已经跟上面的内存地址重复了
(lldb) p/x 0x000000010036a0f0 & 0x00007ffffffffff8
(long) $14 = 0x000000010036a0f0
(lldb) po 0x000000010036a0f0
NSObject
```

到这里，是不是看明白了点啥？`NSObject`类对象也有指向`NSObject`的元类，`Person`的元类的`isa`指向的是`NSObject`的元类。

### 1.5 使用相同的办法查看Teacher的isa

```
// 这里使用简单的方式，直接使用Teacher类
(lldb) x/4gx Teacher.class
0x100008380: 0x00000001000083a8 0x00000001000083f8
0x100008390: 0x0000000100362370 0x0000803000000000

(lldb) p/x 0x00000001000083a8 & 0x00007ffffffffff8
(long) $18 = 0x00000001000083a8

(lldb) po 0x00000001000083a8
Teacher

(lldb) x/4gx 0x00000001000083a8
0x1000083a8: 0x000000010036a0f0 0x00000001000083d0
0x1000083b8: 0x0000000101138140 0x0001e03100000003

(lldb) p/x 0x000000010036a0f0 & 0x00007ffffffffff8
(long) $20 = 0x000000010036a0f0
// 这里又指向了NSObject
(lldb) po 0x000000010036a0f0
NSObject
```

看到这里应该发现了点东西吧。实例对象的isa->类对象的isa->NSObject的isa，中间类对象与继承没有一丢丢关系。

### 继承链



## 1.6 总结

![](isa_metaclass.png)

* 每个实例对象的isa指针指向与之对应的类对象(Class)。
* 每个类对象(Class)都有一个isa指针指向一个唯一的元类(Meta Class)。
* 每一个元类(Meta Class)的isa指针都指向最上层的元类(Meta Class)（图中的NSObject的Meta Class）。最上层的元类(Meta Class)的isa指针指向自己，形成一个回路。
* 每一个元类(Meta Class)的Super Class指向它原本Class的Super Class的Meta Class。最上层的Meta Class的Super Class指向NSObject Class本身。
* 最上层的NSObject Class的Super Class指向nil。
* 只有Class才有继承关系，实例对象与实例对象不存在继承关系。
* 每一个类对象(Class)在内存中都只有一份。


# 2. 通过源码分析

接下来我们从runtime的源码上分析这些都是什么东西。

## 2.1 实例对象 id（Instance）

```
/// A pointer to an instance of a class.
typedef struct objc_object *id;
```
id 这个struct的定义本身就带了 个 ＊, 所以我们在使用其他NSObject类型的实例时需要在前加上 ＊, 使 id 时却不用 。

什么是objc_object?

``` 
/// Represents an instance of a class. 
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
}; 
```
这个时候我们知道Objective-C中的object在最后会被转换成C的结构体, 在这个struct中有 个 isa 指针,指向它的类别 Class。 

## 2.2 类对象 Class

```
/// An opaque type that represents an Objective-C class.
typedef struct objc_class *Class;
```
Class的本质就是一个`objc_class`的结构体。

```
// 注意，这个源码是被简化之后的。
struct objc_class : objc_object {
  
    // Class ISA;
    Class superclass;
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags

    Class getSuperclass() const {
        return superclass;
    }

    void setSuperclass(Class newSuperclass) {
        superclass = newSuperclass;
    }

    class_rw_t *data() const {
        return bits.data();
    }
    void setData(class_rw_t *newData) {
        bits.setData(newData);
    }

    bool isRootClass() {
        return getSuperclass() == nil;
    }
    bool isRootMetaclass() {
        return ISA() == (Class)this;
    }
}
```

看到这个结构体，大家可能会觉得不对，这个源码是错的，不是我们经常看到的，里头没有那些我们常说的变量，methodLists、ivars等等。
大家看仔细了哦，下面这个实现基本都是大家常看到的：

```
struct objc_class {  
    ...
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
} OBJC2_UNAVAILABLE

```
里头确实有ivars、methodLists等，但是这个是`OBJC2_UNAVAILABLE`。其内部确实有这些东西的，我们一步步去探究。

继续回到上面的结构体，发现ISA变量被注释掉了，其实也没有影响的，因为`objc_class` 继承自 `objc_object`（内部有isa变量）。那我们的属性、方法是存放在哪了呢？

通过查看源码，我们看到有这么一个属性`class_data_bits_t`，这个东西里可能存放着我们需要的东西。稍后我们做验证。

## 2.3 元类 Meta Class
OC中一切皆为对象
Class在设计中本身也是一个对象,也有superclass。而这个Class对应的类我们叫“元类”（Meta Class）。也就是说Class中有一个isa指向的是Meta Class。

## 2.4 验证属性的存在位置

在2.2小结我们说了属性、方法、等可能存在于`class_data_bits_t`这个结构体内部，我们查看它的源码：

```
struct class_data_bits_t {
    friend objc_class;

    // Values are the FAST_ flags above.
    uintptr_t bits;
    
public:

    class_rw_t* data() const {
        return (class_rw_t *)(bits & FAST_DATA_MASK);
    }
    void setData(class_rw_t *newData)
    {
        ASSERT(!data()  ||  (newData->flags & (RW_REALIZING | RW_FUTURE)));
        // Set during realization or construction only. No locking needed.
        // Use a store-release fence because there may be concurrent
        // readers of data and data's contents.
        uintptr_t newBits = (bits & ~FAST_DATA_MASK) | (uintptr_t)newData;
        atomic_thread_fence(memory_order_release);
        bits = newBits;
    }
};
```

在public的方法中有`class_rw_t* data()`这个方法，我们进一步探索：


```
struct class_rw_t {
    // Be warned that Symbolication knows the layout of this structure.
    uint32_t flags;
    uint16_t witness;
#if SUPPORT_INDEXED_ISA
    uint16_t index;
#endif

    explicit_atomic<uintptr_t> ro_or_rw_ext;

    Class firstSubclass;
    Class nextSiblingClass;

    const method_array_t methods() const {
        auto v = get_ro_or_rwe();
        if (v.is<class_rw_ext_t *>()) {
            return v.get<class_rw_ext_t *>(&ro_or_rw_ext)->methods;
        } else {
            return method_array_t{v.get<const class_ro_t *>(&ro_or_rw_ext)->baseMethods()};
        }
    }

    const property_array_t properties() const {
        auto v = get_ro_or_rwe();
        if (v.is<class_rw_ext_t *>()) {
            return v.get<class_rw_ext_t *>(&ro_or_rw_ext)->properties;
        } else {
            return property_array_t{v.get<const class_ro_t *>(&ro_or_rw_ext)->baseProperties};
        }
    }

    const protocol_array_t protocols() const {
        auto v = get_ro_or_rwe();
        if (v.is<class_rw_ext_t *>()) {
            return v.get<class_rw_ext_t *>(&ro_or_rw_ext)->protocols;
        } else {
            return protocol_array_t{v.get<const class_ro_t *>(&ro_or_rw_ext)->baseProtocols};
        }
    }
}
```

确实如我们所说的，这里确实存在着我们想要的东西：methods、properties、protocols等。

那我们该怎么获取到这些数据，来证明这些就是我们想要的东西呢？

我们知道在c语言中，一个数组，获取数组中的某个元素的值有多种方法：

```
int a[] = {1,2,3};
printf("index 1 = %d - %d", a[1], *(a+1));
```

比如上面的代码，我们可以直接输出某个元素的下标，也可以通过内存地址来偏移进行读取，同样，我们也可以采取地址偏移来获取`objc_class->bits`的值。

需要偏移多少呢？

第一个变量是Class，这是一个结构体，内部有一个isa指针，所以这是8个字节。
第二个变量是cache_t，我们进源码看一下：

```
struct cache_t {
private:
    explicit_atomic<uintptr_t> _bucketsAndMaybeMask;    // 8
    union {
        struct {
            explicit_atomic<mask_t>    _maybeMask;      // 4
#if __LP64__
            uint16_t                   _flags;          // 2
#endif
            uint16_t                   _occupied;       // 2
        };
        explicit_atomic<preopt_cache_t *> _originalPreoptCache; // 8
    };
...   
... 

};
```
其实这些就能算出来我们需要多少字节，我已经标好了。静态变量和方法是没有算在结构体内部的哈。

所以8+8+8 = 32个字节。

也就是我们获取到的`objc_class`的isa指针，然后偏移32个字节，也就是`0x20`。

我们做一下验证。

### 2.4.1 lldb 验证属性存放的位置

```
(lldb) po p
<Person: 0x10060e160>

(lldb) x/4gx p
0x10060e160: 0x021d8001000081cd 0x0000000000000000
0x10060e170: 0x0000000000000000 0x86c8f7c495bce30f

// 地址0x10060e160+0x20(偏移32个字节)
(lldb) p/x (class_data_bits_t *)0x10060e180
(class_data_bits_t *) $3 = 0x000000010060e180

(lldb) p $3->data()
(class_rw_t *) $4 = 0x0000000100605fd0

(lldb) p *$4
(class_rw_t) $5 = {
  flags = 3592064
  witness = 1
  ro_or_rw_ext = {
    std::__1::atomic<unsigned long> = {
      Value = 4301317328
    }
  }
  firstSubclass = 0x000000010060e4d8
  nextSiblingClass = 0x000000010060e4d8
}

(lldb) p $4->properties()
(const property_array_t) $8 = {
  list_array_tt<property_t, property_list_t, RawPtr> = {
     = {
      list = {
        ptr = 0x0000000100606a50
      }
      arrayAndFlag = 4301285968
    }
  }
}
```

进一步的验证还有待去探索，先到这里，之后补充。


# 2. 属性和成员变量

我们在
```
@interface Person : NSObject
{
    NSString *hobby;
}

@property (nonatomic, copy) NSString *name;

+ (void)sayHello;
- (void)sayHi;

@end
```

通过clang，将main.m生成cpp文件查看类中两者的区别。

```
typedef struct objc_object Person;

struct Person_IMPL {
	struct NSObject_IMPL NSObject_IVARS;
	NSString *hobby;
  // 声明的属性name
	NSString *_name;
};

// @property (nonatomic, copy) NSString *name;
// + (void)sayHello;
// - (void)sayHi;

/* @end */

// @implementation Person
static void _C_Person_sayHello(Class self, SEL _cmd) {
}

static void _I_Person_sayHi(Person * self, SEL _cmd) {
}

static NSString * _I_Person_name(Person * self, SEL _cmd) { return (*(NSString **)((char *)self + OBJC_IVAR_$_Person$_name)); }
extern "C" __declspec(dllimport) void objc_setProperty (id, SEL, long, id, bool, bool);

static void _I_Person_setName_(Person * self, SEL _cmd, NSString *name) { objc_setProperty (self, _cmd, __OFFSETOFIVAR__(struct Person, _name), (id)name, 0, 1); }
// @end
```

我们发现，声明的属性name，被直接转化成了带有下划线的成员变量。同时在实现中，多了set、get方法。

1. 属性： 在cpp文件中属性有下划线，并且自动生成set和get方法
2. 成员变量：没有下划线，没有set、get方法
3. 实例变量：特殊的的成员变量（类的实例化）



# 3. 方法

我们在类中声明的方法都在`method_list_t`中，发现这里并没有我们的类方法。

```
static struct /*_method_list_t*/ {
	unsigned int entsize;  // sizeof(struct _objc_method)
	unsigned int method_count;
	struct _objc_method method_list[5];
} _OBJC_$_INSTANCE_METHODS_Person __attribute__ ((used, section ("__DATA,__objc_const"))) = {
	sizeof(_objc_method),
	5,
	{{(struct objc_selector *)"sayHi", "v16@0:8", (void *)_I_Person_sayHi},
	{(struct objc_selector *)"name", "@16@0:8", (void *)_I_Person_name},
	{(struct objc_selector *)"setName:", "v24@0:8@16", (void *)_I_Person_setName_},
	{(struct objc_selector *)"name", "@16@0:8", (void *)_I_Person_name},
	{(struct objc_selector *)"setName:", "v24@0:8@16", (void *)_I_Person_setName_}}
};
```
因为类方法在元类里头。
类方法的声明
```
static struct /*_method_list_t*/ {
	unsigned int entsize;  // sizeof(struct _objc_method)
	unsigned int method_count;
	struct _objc_method method_list[1];
} _OBJC_$_CLASS_METHODS_Person __attribute__ ((used, section ("__DATA,__objc_const"))) = {
	sizeof(_objc_method),
	1,
	{{(struct objc_selector *)"sayHello", "v16@0:8", (void *)_C_Person_sayHello}}
};
```

这里我们以name的get方法为例子，说明一下这都是什么意思：

```
- (NSString *)getName;
{(struct objc_selector *)"name", "@16@0:8", (void *)_I_Person_name}
```

![](function_table.png)


> @16@0:8
* '@'：第一个@表示返回值，对象
* '16'：16个字节
* '@'：第二个@表示对象类型(id)
* '0'：我们知道@表示对象，0表示从0开始，占8个字节
* ':'：SEL，方法明
* '8'：表示从8开始，占8个字节，满足一共16个字节
 

sel 和 imp
sel：方法名
imp：方法实现。函数指针地址


# 总结

* 实例对象、类对象、元类。一切皆为对象，主要因为objc_object这个结构体。
* isa的走位图，superClass的指向
* 属性、变量，实例方法、类方法的存放
    * 属性、变量的区别，存放的位置
    * 实例方法放在类对象的列表；类方法的存放在元类的方法列表
* sel、imp的区别



```
(lldb) x/6gx LGPerson.class
0x1000083f8: 0x00000001000083d0 0x000000010036a140
0x100008408: 0x00000001006176b0 0x0001803000000003
0x100008418: 0x00000001012042e4 0x00000001000b9970
(lldb) p (class_data_bits_t *)0x100008418
(class_data_bits_t *) $1 = 0x0000000100008418
(lldb) p $1->data()
(class_rw_t *) $2 = 0x00000001012042e0
(lldb) p *$2
(class_rw_t) $4 = {
  flags = 2156396544
  witness = 1
  ro_or_rw_ext = {
    std::__1::atomic<unsigned long> = {
      Value = 4295000480
    }
  }
  firstSubclass = LGTeacher
  nextSiblingClass = NSBinder
}
(lldb) p $2->properties()
(const property_array_t) $5 = {
  list_array_tt<property_t, property_list_t, RawPtr> = {
     = {
      list = {
        ptr = 0x0000000100008320
      }
      arrayAndFlag = 4295000864
    }
  }
}
(lldb) p $5.list
(const RawPtr<property_list_t>) $6 = {
  ptr = 0x0000000100008320
}
(lldb) p $6.ptr
(property_list_t *const) $7 = 0x0000000100008320
(lldb) p *$7
(property_list_t) $8 = {
  entsize_list_tt<property_t, property_list_t, 0, PointerModifierNop> = (entsizeAndFlags = 16, count = 2)
}
(lldb) p $8.get(0)
(property_t) $9 = (name = "name", attributes = "T@\"NSString\",C,N,V_name")
(lldb) p $8.get(1)
(property_t) $10 = (name = "age", attributes = "Ti,N,V_age")
(lldb) 


ivars

(lldb) p $1->safe_ro()
(const class_ro_t *) $11 = 0x00000001000081a0
(lldb) p *$11
(const class_ro_t) $12 = {
  flags = 0
  instanceStart = 8
  instanceSize = 40
  reserved = 0
   = {
    ivarLayout = 0x0000000000000000
    nonMetaclass = nil
  }
  name = {
    std::__1::atomic<const char *> = "LGPerson" {
      Value = 0x0000000100003edc "LGPerson"
    }
  }
  baseMethodList = 0x00000001000081e8
  baseProtocols = 0x0000000000000000
  ivars = 0x0000000100008298
  weakIvarLayout = 0x0000000000000000
  baseProperties = 0x0000000100008320
  _swiftMetadataInitializer_NEVER_USE = {}
}
(lldb) p $11.ivars
(const ivar_list_t *const) $13 = 0x0000000100008298
  Fix-it applied, fixed expression was: 
    $11->ivars
(lldb) p $11->ivars
(const ivar_list_t *const) $14 = 0x0000000100008298
(lldb) p *$14
(const ivar_list_t) $15 = {
  entsize_list_tt<ivar_t, ivar_list_t, 0, PointerModifierNop> = (entsizeAndFlags = 32, count = 4)
}
(lldb) p $15.get(0)
(ivar_t) $16 = {
  offset = 0x0000000100008360
  name = 0x0000000100003f06 "_hobby"
  type = 0x0000000100003f63 "@\"NSString\""
  alignment_raw = 3
  size = 8
}
(lldb) p $15.get(1)
(ivar_t) $17 = {
  offset = 0x0000000100008368
  name = 0x0000000100003f0d "_height"
  type = 0x0000000100003f6f "d"
  alignment_raw = 3
  size = 8
}
(lldb) p $15.get(2)
(ivar_t) $18 = {
  offset = 0x0000000100008370
  name = 0x0000000100003f15 "_age"
  type = 0x0000000100003f71 "i"
  alignment_raw = 2
  size = 4
}
(lldb) p $15.get(3)
(ivar_t) $19 = {
  offset = 0x0000000100008378
  name = 0x0000000100003f1a "_name"
  type = 0x0000000100003f63 "@\"NSString\""
  alignment_raw = 3
  size = 8
}
(lldb) 


实例方法


(lldb) p $1->methods()
error: <user expression 22>:1:5: no member named 'methods' in 'class_data_bits_t'
$1->methods()
~~  ^
(lldb) p $2->methods()
(const method_array_t) $20 = {
  list_array_tt<method_t, method_list_t, method_list_t_authed_ptr> = {
     = {
      list = {
        ptr = 0x00000001000081e8
      }
      arrayAndFlag = 4295000552
    }
  }
}
(lldb) p $20.list
(const method_list_t_authed_ptr<method_list_t>) $21 = {
  ptr = 0x00000001000081e8
}
(lldb) p $21.ptr
(method_list_t *const) $22 = 0x00000001000081e8
(lldb) p *$22
(method_list_t) $23 = {
  entsize_list_tt<method_t, method_list_t, 4294901763, method_t::pointer_modifier> = (entsizeAndFlags = 27, count = 7)
}
(lldb) p $22.get(0)
error: Execution was interrupted, reason: EXC_BAD_ACCESS (code=1, address=0x4000000004).
The process has been returned to the state before expression evaluation.
(lldb) p $23.get(0)
(method_t) $24 = {}
(lldb) p $23.get(0).big()
(method_t::big) $25 = {
  name = "saySomething"
  types = 0x0000000100003f5b "v16@0:8"
  imp = 0x0000000100003c60 (KCObjcBuild`-[LGPerson saySomething])
}
(lldb) p $23.get(1).big()
(method_t::big) $26 = {
  name = "func1"
  types = 0x0000000100003f5b "v16@0:8"
  imp = 0x0000000100003c90 (KCObjcBuild`-[LGPerson func1])
}
(lldb) p $23.get(2).big()
(method_t::big) $27 = {
  name = "func2"
  types = 0x0000000100003f5b "v16@0:8"
  imp = 0x0000000100003cc0 (KCObjcBuild`-[LGPerson func2])
}
(lldb) p $23.get(3).big()
(method_t::big) $28 = {
  name = "name"
  types = 0x0000000100003f53 "@16@0:8"
  imp = 0x0000000100003cf0 (KCObjcBuild`-[LGPerson name])
}
(lldb) p $23.get(4).big()
(method_t::big) $29 = {
  name = "setName:"
  types = 0x0000000100003f73 "v24@0:8@16"
  imp = 0x0000000100003d20 (KCObjcBuild`-[LGPerson setName:])
}
(lldb) p $23.get(5).big()
(method_t::big) $30 = {
  name = "age"
  types = 0x0000000100003f7e "i16@0:8"
  imp = 0x0000000100003d50 (KCObjcBuild`-[LGPerson age])
}
(lldb) p $23.get(6).big()
(method_t::big) $31 = {
  name = "setAge:"
  types = 0x0000000100003f86 "v20@0:8i16"
  imp = 0x0000000100003d70 (KCObjcBuild`-[LGPerson setAge:])
}
(lldb) p $23.get(7).big()
Assertion failed: (i < count), function get, file /Users/alan/Desktop/Learn/source_code/objc4-818.2/runtime/objc-runtime-new.h, line 624.
error: Execution was interrupted, reason: signal SIGABRT.
The process has been returned to the state before expression evaluation.
(lldb) 



类方法
(lldb) x/4gx LGPerson.class
0x1000083f8: 0x00000001000083d0 0x000000010036a140
0x100008408: 0x00000001006176b0 0x0001803000000003
(lldb) p 0x00000001000083d0 & 0x00007ffffffffff8
(long) $33 = 4295001040
(lldb) po $33
LGPerson

(lldb) x/6gx $33
0x1000083d0: 0x000000010036a0f0 0x000000010036a0f0
0x1000083e0: 0x00000001006959d0 0x0002e03100000003
0x1000083f0: 0x0000000101204304 0x00000001000083d0
(lldb) p (class_data_bits_t *)0x1000083f0
(class_data_bits_t *) $34 = 0x00000001000083f0
(lldb) p $34->data()
(class_rw_t *) $35 = 0x0000000101204300
(lldb) *$35
error: '*$35' is not a valid command.
(lldb) p *$35
(class_rw_t) $36 = {
  flags = 2684878849
  witness = 1
  ro_or_rw_ext = {
    std::__1::atomic<unsigned long> = {
      Value = 4302330705
    }
  }
  firstSubclass = 0x00000001000083a8
  nextSiblingClass = 0x00007fff883ac410
}
(lldb) p $35->methods()
(const method_array_t) $37 = {
  list_array_tt<method_t, method_list_t, method_list_t_authed_ptr> = {
     = {
      list = {
        ptr = 0x0000000100008168
      }
      arrayAndFlag = 4295000424
    }
  }
}
(lldb) p $37.list
(const method_list_t_authed_ptr<method_list_t>) $38 = {
  ptr = 0x0000000100008168
}
(lldb) p $38.ptr
(method_list_t *const) $39 = 0x0000000100008168
(lldb) p *$39
(method_list_t) $40 = {
  entsize_list_tt<method_t, method_list_t, 4294901763, method_t::pointer_modifier> = (entsizeAndFlags = 27, count = 2)
}
(lldb) p $40.get(0).big()
(method_t::big) $41 = {
  name = "func3"
  types = 0x0000000100003f5b "v16@0:8"
  imp = 0x0000000100003c00 (KCObjcBuild`+[LGPerson func3])
}
(lldb) p $40.get(1).big()
(method_t::big) $42 = {
  name = "func4"
  types = 0x0000000100003f5b "v16@0:8"
  imp = 0x0000000100003c30 (KCObjcBuild`+[LGPerson func4])
}
(lldb) p $40.get(2).big()
Assertion failed: (i < count), function get, file /Users/alan/Desktop/Learn/source_code/objc4-818.2/runtime/objc-runtime-new.h, line 624.
error: Execution was interrupted, reason: signal SIGABRT.
The process has been returned to the state before expression evaluation.
(lldb) 
```