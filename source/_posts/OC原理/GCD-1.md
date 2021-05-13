---
title: GCD-1
date: 2021-05-10 21:24:11
tags:
    - Objective-C,
    - iOS
categories:
    - OC原理
---

# GCD简介

全称是 Grand Central Dispatch。底层为C语言，将任务添加到队列，并且指定执行任务的函数。GCD提供了非常强大的函数。

## GCD的优势

* 是苹果公司为多核的并行运算提出的解决方案
* 会自动利用更多的CPU内核(比如双核、四核)
* 会自动管理线程的生命周期(创建线程、调度任务、销毁线程) 程序员只需要告诉 GCD 想要执行什么任务，不需要编写任何线程管理代码

## 同步和异步函数

GCD使用block封装任务，任务的block没有参数也没有返回值。

任务的调度有同步和异步之分。

### 同步`dispatch_sync`

1. 必须等待当前语句执行完毕，才会执行下一条语句。
2. 同步不会开启线程。
3. 在当前线程执行block任务。

### 异步 `dispatch_async`

1. 会开启新线程执行block任务
2. 异步是多线程的代名词

## 队列

队列是一种数据结构，遵循先进先出的原则（FIFO）。分为串行队列和并发队列（并行）。

不管是串行还是并发队列，谁在队列的最前头谁先开始执行。但是执行的快慢与当前所需资源有关。

![](queue.jpg)

串行等待上一个任务执行完成
并发不会等待上一个任务执行完成


## 函数与队列

### 同步函数 + 串行队列

* 不会开启线程，在当前线程执行任务
* 执行完一个执行下一个，会产生堵塞

```
// 创建串行队列
- (void)serialSyncTest{
    dispatch_queue_t queue = dispatch_queue_create("queue1", DISPATCH_QUEUE_SERIAL);
    for (int i = 0; i < 20; i++) {
        // a
        dispatch_sync(queue, ^{
            // b
            NSLog(@"i = %d, thread = %@",i,[NSThread currentThread]);
        });
        // c
    }
}
```

会从0到19，按照a-b-c的顺序输出所有数据。根据打印的线程，发现就是主线程，并不会开启新线程。
    
### 同步函数 + 并发队列

* 不会开启线程，在当前线程执行任务
* 任务一个接着一个执行

```
- (void)concurrentSyncTest{

    //1:创建并发队列
    dispatch_queue_t queue = dispatch_queue_create("queue2", DISPATCH_QUEUE_CONCURRENT);
    for (int i = 0; i<20; i++) {
        dispatch_sync(queue, ^{
            NSLog(@"i = %d, thread = %@",i,[NSThread currentThread]);
        });
    }
    NSLog(@"hello queue");
}
```

同步并发队列，会按照顺序执行，最后打印`hello queue`。

同步函数的情况下，不管是串行还是并发，都不会开启新线程，任务按步执行。

    
### 异步函数 + 串行队列

* 开启新线程
* 任务一个接一个执行

```
// 异步串行
- (void)serialAsyncTest{
    //1:创建串行队列
    dispatch_queue_t queue = dispatch_queue_create("queue3", DISPATCH_QUEUE_SERIAL);
    for (int i = 0; i<20; i++) {
        // a
        dispatch_async(queue, ^{
            // b
            NSLog(@"i = %d，thread = %@",i,[NSThread currentThread]);
        });
        // c
    }
    // d
    NSLog(@"hello queue");
}
```

由于是异步串行队列，线程的创建会有耗时操作，在for循环中执行的顺序是a-c-b(a)，执行了a之后是c，在之后不一定是a还是b。

而d语句可能先执行，也可能后执行。

根据线程的打印情况，发现会开启新线程。
    
### 异步函数 + 并发队列

* 开启新线程，并开始执行
* 任务异步执行，没有顺序，与CPU调度有关

```
- (void)concurrentAsyncTest{
    //1:创建并发队列
    dispatch_queue_t queue = dispatch_queue_create("queue4", DISPATCH_QUEUE_CONCURRENT);
    for (int i = 0; i<20; i++) {
        dispatch_async(queue, ^{
            NSLog(@"%d-%@",i,[NSThread currentThread]);
        });
    }
    NSLog(@"hello queue");
}
```

会开启新的线程，执行没有顺序。


## 函数队列的面试题

### 异步并发队列
```
- (void)textDemo {
    dispatch_queue_t queue = dispatch_queue_create("queue", DISPATCH_QUEUE_CONCURRENT);
    NSLog(@"1");
    // 耗时
    dispatch_async(queue, ^{
        NSLog(@"2");
        dispatch_async(queue, ^{
            NSLog(@"3");
        });
        NSLog(@"4");
    });
    NSLog(@"5");
}
```

这是一个并行队列，内部是异步执行。当block内部有任务需要执行时，会产生耗时，所以就会先执行完成block外部的简单调用。而在block内部，是按照正常的流程执行的。

打印的结果是`1 5 2 4 3`。

### 异步串行队列

```
- (void)textDemo1{
    // 串行队列
    dispatch_queue_t queue = dispatch_queue_create("queue", NULL);
    NSLog(@"1");
    dispatch_async(queue, ^{
        NSLog(@"2");
        dispatch_async(queue, ^{
            NSLog(@"3");
        });
        NSLog(@"4");
    });
    NSLog(@"5");
}
```

只要是block内部需要执行，则一定是耗时操作，所以先执行1，然后5，在异步串行队列内部，是与外部一样的道理，

结果是：1 5 2 4 3

### 并发队列 异步同步嵌套

```
- (void)textDemo3 {
    // 并发队列
    dispatch_queue_t queue = dispatch_queue_create("queue", DISPATCH_QUEUE_CONCURRENT);
    NSLog(@"1");
    dispatch_async(queue, ^{
        NSLog(@"2");
        dispatch_sync(queue, ^{
            NSLog(@"3");
        });
        NSLog(@"4");
    });
    NSLog(@"5");
}
```

结果为 1 5 2 3 4
同步并发队列也不会开启新线程，一个一个执行。

### 串行 异步同步嵌套

```
- (void)textDemo4 {
    // 串行队列
    dispatch_queue_t queue = dispatch_queue_create("queue", NULL);
    NSLog(@"1");
    // 异步函数
    dispatch_async(queue, ^{
        NSLog(@"2");
        // 同步
        dispatch_sync(queue, ^{
            NSLog(@"3");
        });
        NSLog(@"4");
    });
    NSLog(@"5");
}
```

上面已经有过异步串行、异步并行的例子了，只要异步函数内部有任务要执行，就属于耗时操作，会优先执行完毕外部的简单操作。所以先执行 1 5

在异步函数内部，继续串行执行。这时候会执行2，然后碰到了同步串行队列。而同步串行队列是需要等待外部执行完成之后才会执行，但是4也在等待同步函数的执行，造成了互相等待，发生了死锁。

![](loop-queue.jpg)

这里即使把4注释掉，也同样会发生死锁。

所以结果为： 1 5 2 -- 死锁

### 总结

我们一定要清楚，不管是异步还是同步，都是对与block块和其下一行代码来说的。在block内部不管当前是异步还是同步，串行还是并行，都是从上往下执行的。

另外并发和串行的区别：
* 并发不会等待一个任务执行完成才执行。
* 串行会等待一个任务执行完毕才执行。

同步和异步：
* 异步函数下，不管是串行队列还是并行队列，都不影响block块之外的内存执。因为block内部是在新开启的线程中执行的。
* 同步函数下，并行队列不受影响，因为并行不需要等待上一个任务执行完成。如果是串行队列，那在当前线程下会发生死锁。


# 主队列 & 全局队列

```
dispatch_queue_t serial = dispatch_queue_create("serial", DISPATCH_QUEUE_SERIAL);
dispatch_queue_t conque = dispatch_queue_create("conque", DISPATCH_QUEUE_CONCURRENT);
dispatch_queue_t mainQueue = dispatch_get_main_queue();
dispatch_queue_t globQueue = dispatch_get_global_queue(0, 0);

NSLog(@"%@\n%@\n%@\n%@",serial,conque,mainQueue,globQueue);
```

打印一些这4个队列：

```
<OS_dispatch_queue_serial: serial>
<OS_dispatch_queue_concurrent: conque>
<OS_dispatch_queue_main: com.apple.main-thread>
<OS_dispatch_queue_global: com.apple.root.default-qos>
```

这里打印了4个队列，但是其实一共只有两个队列，就是串行队列和并发队列。

通过汇编手法，我们发现GCD的源码存在与`libdispatch.dylib`库中，我们就从这个库里看GCD的底层实现。

## 主队列

`dispatch_get_main_queue()`主队列专门用来在主线程上调度任务的<b>串行队列</b>，并不会开启新线程。

如果当前主线程正在执行任务，那么无论主队列中被添加了什么任务，都不会被调度执行。

```
dispatch_queue_t mainQueue = dispatch_get_main_queue();
```

我们通过源码查看

```
//
dispatch_queue_main_t
dispatch_get_main_queue(void)
{
	return DISPATCH_GLOBAL_OBJECT(dispatch_queue_main_t, _dispatch_main_q);
}
```
在`dispatch_get_main_queue`的解释中，我们发现：主队列依赖于主线程`dispatch_main()`和`runloop`，并且主线程是在`main()`函数之前自动创建的（dyld的流程）。

先看看啥是`dispatch_queue_main_t`

> A dispatch queue that is bound to the app's main thread and executes tasks serially on that thread.

```
typedef NSObject<OS_dispatch_queue_main> *dispatch_queue_main_t;
```

可以看出来`OS_dispatch_queue_main`是一个类。


那我们找一找`DISPATCH_GLOBAL_OBJECT`这个的实现：

```
#define DISPATCH_GLOBAL_OBJECT(type, object) ((OS_OBJECT_BRIDGE type)&(object))
```

这是一个宏定义，内部使用的是一个type类型强转之后与object进行二进制的”&“运算。

然后看看这两个参数：

`dispatch_queue_main_t`：
> The type of the default queue that is bound to the main thread
从字面意思就是把默认线程绑定到主线程。

`_dispatch_main_q`：
> Returns the default queue that is bound to the main thread.
返回一个绑定了主线程的默认线程。接下来我们通过源码看一下`_dispatch_main_q`。

```
struct dispatch_queue_static_s _dispatch_main_q = {
	DISPATCH_GLOBAL_OBJECT_HEADER(queue_main),
#if !DISPATCH_USE_RESOLVERS
	.do_targetq = _dispatch_get_default_queue(true),
#endif
	.dq_state = DISPATCH_QUEUE_STATE_INIT_VALUE(1) |
			DISPATCH_QUEUE_ROLE_BASE_ANON,
	.dq_label = "com.apple.main-thread",
	.dq_atomic_flags = DQF_THREAD_BOUND | DQF_WIDTH(1),
	.dq_serialnum = 1,
};
```

`_dispatch_main_q`是一个结构体：

`dq_label`：使用的标签，上方代码中打印出来的东西。
`dq_atomic_flags`：是一个flag，DQF_WIDTH(1)表示宽度，1只能通过1个
`dq_serialnum`：串行数是1


知道了两个参数，我们直接使用”&“运算看是否能得到我们想要的主线程。

```
dispatch_queue_t mainQueue = (OS_OBJECT_BRIDGE dispatch_queue_main_t)&(_dispatch_main_q);
```

得到的这个mainQueue与上方的点结果是一直到。

接下来我们得验证一下，`dispatch_get_main_queue`是在main函数之前执行的。在dyld的流程中，我们知道他会执行一个`libdispatch_init(void)`的操作。在它的内部源码中有如下内容：

```
void
libdispatch_init(void)
{
// ...
// line 7921
#if DISPATCH_USE_RESOLVERS // rdar://problem/8541707
	_dispatch_main_q.do_targetq = _dispatch_get_default_queue(true);
#endif
  // 设置当前线程
	_dispatch_queue_set_current(&_dispatch_main_q);
	// 绑定线程
	_dispatch_queue_set_bound_thread(&_dispatch_main_q);
// ...
}
```

在第7758行代码，可以看到创建了`	_dispatch_main_q`静态结构体，之后设置当前线程为为主线程，然后进行绑定。

## 全局队列

`dispatch_get_global_queue(0,0)`，为了方便使用，苹果创建了全局队列，全局队列是一个<b>并发队列</b>。

在使用多线程开发时，如果对队列没有特殊需求，在执行异步任务时，可以直接使用全局队列。

```
dispatch_queue_global_t
dispatch_get_global_queue(intptr_t identifier, uintptr_t flags);
```

这里有两个参数：
第一个identifier：表示优先级，与QOS的优先级一一对应。

```
DISPATCH_QUEUE_PRIORITY_HIGH        // 高
DISPATCH_QUEUE_PRIORITY_DEFAULT     // 默认
DISPATCH_QUEUE_PRIORITY_LOW         // 低
DISPATCH_QUEUE_PRIORITY_BACKGROUND  // BACKGROUND
```

第二个参数是flag：
保留供将来使用的标志。始终将此参数指定为0。

接下来，查看一下源码：

```
dispatch_queue_global_t
dispatch_get_global_queue(long priority, unsigned long flags)
{
	dispatch_assert(countof(_dispatch_root_queues) ==
			DISPATCH_ROOT_QUEUE_COUNT);

	if (flags & ~(unsigned long)DISPATCH_QUEUE_OVERCOMMIT) {
		return DISPATCH_BAD_INPUT;
	}
	dispatch_qos_t qos = _dispatch_qos_from_queue_priority(priority);
#if !HAVE_PTHREAD_WORKQUEUE_QOS
	if (qos == QOS_CLASS_MAINTENANCE) {
		qos = DISPATCH_QOS_BACKGROUND;
	} else if (qos == QOS_CLASS_USER_INTERACTIVE) {
		qos = DISPATCH_QOS_USER_INITIATED;
	}
#endif
	if (qos == DISPATCH_QOS_UNSPECIFIED) {
		return DISPATCH_BAD_INPUT;
	}
	return _dispatch_get_root_queue(qos, flags & DISPATCH_QUEUE_OVERCOMMIT);
}
```

这一坨东西其实都不用看，只需要看到最后`return _dispatch_get_root_queue()`是这么个东西。

```
static inline dispatch_queue_global_t
_dispatch_get_root_queue(dispatch_qos_t qos, bool overcommit)
{
	if (unlikely(qos < DISPATCH_QOS_MIN || qos > DISPATCH_QOS_MAX)) {
		DISPATCH_CLIENT_CRASH(qos, "Corrupted priority");
	}
	return &_dispatch_root_queues[2 * (qos - 1) + overcommit];
}
```

`_dispatch_root_queues[]`应该就是一个数组，通过传进来的参数获取对应的queue。

```
struct dispatch_queue_global_s _dispatch_root_queues[] = {
// ...
  .dq_atomic_flags = DQF_WIDTH(DISPATCH_QUEUE_WIDTH_POOL), \
	_DISPATCH_ROOT_QUEUE_ENTRY(MAINTENANCE, 0,
		.dq_label = "com.apple.root.maintenance-qos",
		.dq_serialnum = 4,
	),
	_DISPATCH_ROOT_QUEUE_ENTRY(MAINTENANCE, DISPATCH_PRIORITY_FLAG_OVERCOMMIT,
		.dq_label = "com.apple.root.maintenance-qos.overcommit",
		.dq_serialnum = 5,
	),
	_DISPATCH_ROOT_QUEUE_ENTRY(BACKGROUND, 0,
		.dq_label = "com.apple.root.background-qos",
		.dq_serialnum = 6,
	),
	_DISPATCH_ROOT_QUEUE_ENTRY(BACKGROUND, DISPATCH_PRIORITY_FLAG_OVERCOMMIT,
		.dq_label = "com.apple.root.background-qos.overcommit",
		.dq_serialnum = 7,
	),
	_DISPATCH_ROOT_QUEUE_ENTRY(UTILITY, 0,
		.dq_label = "com.apple.root.utility-qos",
		.dq_serialnum = 8,
	),
	_DISPATCH_ROOT_QUEUE_ENTRY(UTILITY, DISPATCH_PRIORITY_FLAG_OVERCOMMIT,
		.dq_label = "com.apple.root.utility-qos.overcommit",
		.dq_serialnum = 9,
	),
	_DISPATCH_ROOT_QUEUE_ENTRY(DEFAULT, DISPATCH_PRIORITY_FLAG_FALLBACK,
		.dq_label = "com.apple.root.default-qos",
		.dq_serialnum = 10,
	),
	_DISPATCH_ROOT_QUEUE_ENTRY(DEFAULT,
			DISPATCH_PRIORITY_FLAG_FALLBACK | DISPATCH_PRIORITY_FLAG_OVERCOMMIT,
		.dq_label = "com.apple.root.default-qos.overcommit",
		.dq_serialnum = 11,
	),
	_DISPATCH_ROOT_QUEUE_ENTRY(USER_INITIATED, 0,
		.dq_label = "com.apple.root.user-initiated-qos",
		.dq_serialnum = 12,
	),
	_DISPATCH_ROOT_QUEUE_ENTRY(USER_INITIATED, DISPATCH_PRIORITY_FLAG_OVERCOMMIT,
		.dq_label = "com.apple.root.user-initiated-qos.overcommit",
		.dq_serialnum = 13,
	),
	_DISPATCH_ROOT_QUEUE_ENTRY(USER_INTERACTIVE, 0,
		.dq_label = "com.apple.root.user-interactive-qos",
		.dq_serialnum = 14,
	),
	_DISPATCH_ROOT_QUEUE_ENTRY(USER_INTERACTIVE, DISPATCH_PRIORITY_FLAG_OVERCOMMIT,
		.dq_label = "com.apple.root.user-interactive-qos.overcommit",
		.dq_serialnum = 15,
	),
};
```

我们看到了lable的内容，有我们刚才打印的那个`com.apple.root.default-qos`。会根据我们设置的优先级返回不同的全局队列。

```
dq_atomic_flags = DQF_WIDTH(DISPATCH_QUEUE_WIDTH_POOL)

#define DISPATCH_QUEUE_WIDTH_FULL			0x1000ull
#define DISPATCH_QUEUE_WIDTH_POOL (DISPATCH_QUEUE_WIDTH_FULL - 1)
```

dq_atomic_flags的值也就是	(0x1000 - 1) = 4095


# dispatch_queue_create 原理

直奔主题，在源码中查看`dispatch_queue_create`方法：

```
dispatch_queue_t
dispatch_queue_create(const char *label, dispatch_queue_attr_t attr)
{
	return _dispatch_lane_create_with_target(label, attr,
			DISPATCH_TARGET_QUEUE_DEFAULT, true);
}
```

这里传了两个参数，第一个是标签，表示创建的队列，第二个标识串行还是并发。

## 串行：DISPATCH_QUEUE_SERIAL

我们看一下源码：

```
#define DISPATCH_QUEUE_SERIAL NULL
```

所以，通常情况下，我们在创建串行队列时，也会使用`NULL`来替换。

## 并发：DISPATCH_QUEUE_CONCURRENT

我们看一下源码实现：

```
#define DISPATCH_QUEUE_CONCURRENT \
		DISPATCH_GLOBAL_OBJECT(dispatch_queue_attr_t, \
		_dispatch_queue_attr_concurrent)
```

这里有一个`DISPATCH_GLOBAL_OBJECT()`函数，在主队列中已经介绍过了（通过&运算）。


`_dispatch_lane_create_with_target`这个函数中，我们发现很长很难懂，那我们就通过多年的编程经验，看它返回的时候一个什么东西，然后看这个是怎么创建的。下面的代码是经过删减的，有需要的自行查看源码。

```
static dispatch_queue_t
_dispatch_lane_create_with_target(const char *label, dispatch_queue_attr_t dqa,
		dispatch_queue_t tq, bool legacy)
{
	// 1. 创建 dqai
	dispatch_queue_attr_info_t dqai = _dispatch_queue_attr_to_info(dqa);
	// ...
	// 2. 创建vtable
	const void *vtable;
	dispatch_queue_flags_t dqf = legacy ? DQF_MUTABLE : 0;
	if (dqai.dqai_concurrent) {
		// OS_dispatch_queue_concurrent
		vtable = DISPATCH_VTABLE(queue_concurrent);
	} else {
		vtable = DISPATCH_VTABLE(queue_serial);
	}
	// ...
	// 3. label赋值
	if (label) {
		const char *tmp = _dispatch_strdup_if_mutable(label);
		if (tmp != label) {
			dqf |= DQF_LABEL_NEEDS_FREE;
			label = tmp;
		}
	}
	// 4. dq alloc分配内存空间
	dispatch_lane_t dq = _dispatch_object_alloc(vtable,
			sizeof(struct dispatch_lane_s)); 
	// 5. dq init操作
	_dispatch_queue_init(dq, dqf, dqai.dqai_concurrent ?
			DISPATCH_QUEUE_WIDTH_MAX : 1, DISPATCH_QUEUE_ROLE_INNER |
			(dqai.dqai_inactive ? DISPATCH_QUEUE_INACTIVE : 0)); // init
  // 6. 对dq进行赋值
	dq->dq_label = label;
	dq->dq_priority = _dispatch_priority_make((dispatch_qos_t)dqai.dqai_qos,
			dqai.dqai_relpri);
	if (overcommit == _dispatch_queue_attr_overcommit_enabled) {
		dq->dq_priority |= DISPATCH_PRIORITY_FLAG_OVERCOMMIT;
	}
	if (!dqai.dqai_inactive) {
		_dispatch_queue_priority_inherit_from_target(dq, tq);
		_dispatch_lane_inherit_wlh_from_target(dq, tq);
	}
	_dispatch_retain(tq);
	dq->do_targetq = tq;
	_dispatch_object_debug(dq, "%s", __func__);
	return _dispatch_trace_queue_create(dq)._dq;
}
```

## 创建dqai

```
dispatch_queue_attr_info_t
_dispatch_queue_attr_to_info(dispatch_queue_attr_t dqa)
{
	dispatch_queue_attr_info_t dqai = { };

	if (!dqa) return dqai;

#if DISPATCH_VARIANT_STATIC
	if (dqa == &_dispatch_queue_attr_concurrent) { // null 默认
		dqai.dqai_concurrent = true;
		return dqai;
	}
#endif
    // ...
    return dqai;
}
```

还记得我们在创建队列时传的参数吗？第一个是label，第二个是串行还是并发。

这个dqai会判断当前是串行还是并发，并对`dqai.dqai_concurrent = true;`进行赋值。

## 创建 vtable

`vtable`会根据当前是串行还是并发进行创建，我们一步一步的追寻`vtable`是什么。


```
#if OS_OBJECT_HAVE_OBJC2
#define DISPATCH_VTABLE(name) DISPATCH_OBJC_CLASS(name)
#define DISPATCH_OBJC_CLASS(name) (&DISPATCH_CLASS_SYMBOL(name))
#define DISPATCH_CLASS_SYMBOL(name) OS_dispatch_##name##_class
#elif
...
#end

```

```
if (dqai.dqai_concurrent) {
	// OS_dispatch_queue_concurrent
	vtable = DISPATCH_VTABLE(queue_concurrent);
} else {
	vtable = DISPATCH_VTABLE(queue_serial);
}
```

通过源码发现，`vtable`就是一个类。最后生成的就是`OS_dispatch_##name##_class`。

`##name##`就是创建`vtable`时的参数，就会生成对应的`OS_dispatch_queue_serial_class`和`OS_dispatch_queue_concurrent_class`。


## label赋值
	
这个就是创建时传入的那个label标签的内容。

## dq alloc分配内存空间

这里执行了alloc操作，开始分配内存空间。

```	
dispatch_lane_t dq = _dispatch_object_alloc(vtable,
			sizeof(struct dispatch_lane_s)); 
			
			
void *
_dispatch_object_alloc(const void *vtable, size_t size)
{
// 这个是在mac下执行
#if OS_OBJECT_HAVE_OBJC1
...
#else
	// 这里分配内存，isa指向
	return _os_object_alloc_realized(vtable, size);
#endif
}
```

真正的alloc操作是在这里执行的。

```
inline _os_object_t
_os_object_alloc_realized(const void *cls, size_t size)
{
	_os_object_t obj;
	dispatch_assert(size >= sizeof(struct _os_object_s));
	// 开辟空间
	while (unlikely(!(obj = calloc(1u, size)))) {
	// 执行的都是likely的操作，所以不会走这里，这里也没有意义，内部是sleep操作
		_dispatch_temporary_resource_shortage();
	}
	// isa指向
	obj->os_obj_isa = cls;
	return obj;
}
```

## dq init操作

alloc之后，执行init操作。

初始化的时候会判断当前要生成并发还是串行队列，并发的话，个数是`DISPATCH_QUEUE_WIDTH_MAX`，串行是1，就是开辟的最大任务数。

## 对dq进行赋值

比如lable标签、overcommit，priority等赋值。同时绑定target。

## 最后return

```
return _dispatch_trace_queue_create(dq)._dq;
```

这里看源码都是最后返回的都是dq对应的数据。

# dispatch_async 源码

我们接下来看一下异步并发队列函数的源码。

```
dispatch_async(conque, ^{
    NSLog(@"12334");
});
```

```
void
dispatch_async(dispatch_queue_t queue, dispatch_block_t block);

void
dispatch_async(dispatch_queue_t dq, dispatch_block_t work)
{
	dispatch_continuation_t dc = _dispatch_continuation_alloc();
	uintptr_t dc_flags = DC_FLAG_CONSUME;
	dispatch_qos_t qos;

	qos = _dispatch_continuation_init(dc, dq, work, 0, dc_flags);
	_dispatch_continuation_async(dq, dc, qos, dc->dc_flags);
}
```

`dispatch_async()`函数内部会执行`_dispatch_continuation_init`，这个是函数中的重点。看一下源码：

```
static inline dispatch_qos_t
_dispatch_continuation_init(dispatch_continuation_t dc,
		dispatch_queue_class_t dqu, dispatch_block_t work,
		dispatch_block_flags_t flags, uintptr_t dc_flags)
{
// 1. work就是外部的block，这里ctxt是对block的一个copy操作
	void *ctxt = _dispatch_Block_copy(work);

	dc_flags |= DC_FLAG_BLOCK | DC_FLAG_ALLOCATED;
	if (unlikely(_dispatch_block_has_private_data(work))) {
    	// 执行的都是likely的操作
    	// dc_flags赋值	
		dc->dc_flags = dc_flags;
		// block赋值到dc_ctxt中
		dc->dc_ctxt = ctxt;
		// will initialize all fields but requires dc_flags & dc_ctxt to be set
		return _dispatch_continuation_init_slow(dc, dqu, flags);
	}
  // 所以会走这里，func可以理解为work的方法名。
	dispatch_function_t func = _dispatch_Block_invoke(work);
	if (dc_flags & DC_FLAG_CONSUME) {
		func = _dispatch_call_block_and_release;
	}
	// 这里又是重点内容
	return _dispatch_continuation_init_f(dc, dqu, ctxt, func, flags, dc_flags);
}
```

我们进一步查看`_dispatch_continuation_init_f`源码：

```
static inline dispatch_qos_t
_dispatch_continuation_init_f(dispatch_continuation_t dc,
		dispatch_queue_class_t dqu, void *ctxt, dispatch_function_t f,
		dispatch_block_flags_t flags, uintptr_t dc_flags)
{
  // 默认优先级0
	pthread_priority_t pp = 0;
	// 设置dc_flags
	dc->dc_flags = dc_flags | DC_FLAG_ALLOCATED;
	// 设置方法名
	dc->dc_func = f;
	// 方法实现，我们知道dispatch_async是没有参数的。
	dc->dc_ctxt = ctxt;
	// 设置优先级
	if (!(flags & DISPATCH_BLOCK_HAS_PRIORITY)) {
		pp = _dispatch_priority_propagate();
	}
	_dispatch_continuation_voucher_set(dc, flags);
	// 对block调用的优先级处理
	return _dispatch_continuation_priority_set(dc, dqu, pp, flags);
}
```
这个也就是dispatch_async的实现。下一章会继续block是如何调用的。


# 总结

1. GCD的介绍
2. 同步、异步函数的介绍
3. 串行队列、并发队列
4. 函数与队列的4种组合，以及面试题
    1. 并发不会等待一个任务执行完成才执行。
    2. 串行会等待一个任务执行完毕才执行。
    3. 异步函数下，不管是串行队列还是并行队列，都不影响block块之外的内存执。因为block内部是在新开启的线程中执行的。
    4. 同步函数下，并行队列不受影响，因为并行不需要等待上一个任务执行完成。如果是串行队列，那在当前线程下会发生死锁。

5. 主队列dispatch_get_main_queue，全局队列dispatch_get_global_queue内部实现
6. dispatch_queue_create创建一个队列的原理
7. dispatch_async内部实现

# 引用

[libdispatch源文件](https://opensource.apple.com/tarballs/libdispatch/)
这里是用的是libdispatch-1271.40.12.tar.gz文件。



