---
title: GCD底层原理-3
date: 2021-05-16 17:41:00
tags:
    - Objective-C,
    - iOS
categories:
    - OC原理
---

# 信号量

先看代码：

```
- (void)semaphore {
    // 全局队列
    dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
    // 初始化一个信号量可以并发执行2个任务
    dispatch_semaphore_t sem = dispatch_semaphore_create(2);
    
    //任务1
    dispatch_async(queue, ^{
        dispatch_semaphore_wait(sem, DISPATCH_TIME_FOREVER);
        NSLog(@"1 start..");
        sleep(1);
        NSLog(@"1 end...");
        dispatch_semaphore_signal(sem);
    });
    
    //任务2
    dispatch_async(queue, ^{
        dispatch_semaphore_wait(sem, DISPATCH_TIME_FOREVER);
        NSLog(@"2 start..");
        sleep(1);
        NSLog(@"2 end...");
        dispatch_semaphore_signal(sem);
    });
    
    //任务3
    dispatch_async(queue, ^{
        dispatch_semaphore_wait(sem, DISPATCH_TIME_FOREVER);
        NSLog(@"3 start..");
        sleep(1);
        NSLog(@"3 end...");
        dispatch_semaphore_signal(sem);
    });
}
```

我们先运行一下这个demo，看输出的结果是什么样子。

```
2021-05-16 17:38:16.018150+0800 GCDDemo[972:7272767] 1 start..
2021-05-16 17:38:16.018154+0800 GCDDemo[972:7272769] 2 start..
2021-05-16 17:38:17.027319+0800 GCDDemo[972:7272767] 1 end...
2021-05-16 17:38:17.027332+0800 GCDDemo[972:7272769] 2 end...
2021-05-16 17:38:17.027556+0800 GCDDemo[972:7272765] 3 start..
2021-05-16 17:38:18.031174+0800 GCDDemo[972:7272765] 3 end...
```

我们也需要看打印的时间点，先执行1、2 start，1s之后执行了1、2 end，紧接着执行3。

如果把信号量的初始值改成1呢？

```
2021-05-16 18:07:48.061949+0800 GCDDemo[1220:7292903] 1 start..
2021-05-16 18:07:49.063438+0800 GCDDemo[1220:7292903] 1 end...
2021-05-16 18:07:49.063639+0800 GCDDemo[1220:7292901] 2 start..
2021-05-16 18:07:50.063905+0800 GCDDemo[1220:7292901] 2 end...
2021-05-16 18:07:50.064202+0800 GCDDemo[1220:7292906] 3 start..
2021-05-16 18:07:51.064940+0800 GCDDemo[1220:7292906] 3 end...
```

结果就是先执行1，然后2，3每次间隔都是1s。

为什么能确定并发执行的个数呢？就是因为其中有两个语句，确定了最大的并发数。

```
// 执行-1操作
dispatch_semaphore_wait(sem, DISPATCH_TIME_FOREVER);
// 执行+1操作
dispatch_semaphore_signal(sem);
```

分别看一下内部实现。

## dispatch_semaphore_create

```
dispatch_semaphore_t
dispatch_semaphore_create(intptr_t value)
{
	dispatch_semaphore_t dsema;
	if (value < 0) {
		return DISPATCH_BAD_INPUT;
	}
   // 初始化信号量结构体  
	dsema = _dispatch_object_alloc(DISPATCH_VTABLE(semaphore),
			sizeof(struct dispatch_semaphore_s));
	dsema->do_next = DISPATCH_OBJECT_LISTLESS;
	dsema->do_targetq = _dispatch_get_default_queue(false);
	// 用来保存初始化的最大并发数
	dsema->dsema_value = value;
	_dispatch_sema4_init(&dsema->dsema_sema, _DSEMA4_POLICY_FIFO);
	dsema->dsema_orig = value;
	return dsema;
}
```

看到了吧，最开始就已经有判断了，如果创建时的参数小于0，直接返回一个`DISPATCH_BAD_INPUT`。

```
#define DISPATCH_BAD_INPUT		((void *_Nonnull)0)
```

知道时啥了吗？就是一个野指针。

那我们接下来看`dispatch_semaphore_wait`函数。


## dispatch_semaphore_wait

```
intptr_t
dispatch_semaphore_wait(dispatch_semaphore_t dsema, dispatch_time_t timeout)
{
	long value = os_atomic_dec2o(dsema, dsema_value, acquire);
	if (likely(value >= 0)) {
		return 0;
	}
	return _dispatch_semaphore_wait_slow(dsema, timeout);
}
```

这里调用了`os_atomic_dec2o`，但是有一个dec，猜测应该是减法操作。

```
#define os_atomic_dec2o(p, f, m) \
		os_atomic_sub2o(p, f, 1, m)

#define os_atomic_sub2o(p, f, v, m) \
		os_atomic_sub(&(p)->f, (v), m)
		
#define os_atomic_sub(p, v, m) \
		_os_atomic_c11_op((p), (v), m, sub, -)
		
#define _os_atomic_c11_op(p, v, m, o, op) \
		({ _os_atomic_basetypeof(p) _v = (v), _r = \
		atomic_fetch_##o##_explicit(_os_atomic_c11_atomic(p), _v, \
		memory_order_##m); (__typeof__(_r))(_r op _v); })
```

一层一层的宏定义，最后可以得出的函数是`atomic_fetch_sub_explicit`。
这是一个C语言的原子类型减的函数。也就是`dsema->dsema_value - 1`。

大于等于0都可以正常执行，否则发生等待。

## dispatch_semaphore_signal

```
intptr_t
dispatch_semaphore_signal(dispatch_semaphore_t dsema)
{
	long value = os_atomic_inc2o(dsema, dsema_value, release);
	if (likely(value > 0)) {
		return 0;
	}
	if (unlikely(value == LONG_MIN)) {
		DISPATCH_CLIENT_CRASH(value,
				"Unbalanced call to dispatch_semaphore_signal()");
	}
	return _dispatch_semaphore_signal_slow(dsema);
}
```

`dispatch_semaphore_signal`与`dispatch_semaphore_wait`正好相反，执行的是加法操作，每次加1。

这里的判断条件是`value > 0`，都可以正常执行，也就是必须有1个才行，否则阻塞线程，一直处于等待状态。

## 总结

`dispatch_semaphore_wait`和`dispatch_semaphore_signal`肯定是成对出现的。


# dispatch_group 调度组

当我们在业务中，需要两个网络请求都返回之后，才能处理某一个业务逻辑时，调度组就能很好的发挥作用，而且用起来很简单。

先看一下简单的代码：

```
- (void)groupDemo {
    dispatch_group_t group = dispatch_group_create();
    dispatch_queue_t globle = dispatch_get_global_queue(0, 0);
    
    // 1.
    dispatch_group_enter(group);
    dispatch_async(globle, ^{
        NSLog(@"1 start ...");
        sleep(3);
        dispatch_group_leave(group);
    });
    // 2.
    dispatch_group_enter(group);
    dispatch_async(globle, ^{
        NSLog(@"2 start ...");
        sleep(3);
        dispatch_group_leave(group);
    });
    
    // 3.
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"3 ...");
    });
}
```

我们执行一下代码，发现3是最后执行的。

如果出现`dispatch_group_enter`比`dispatch_group_leave`多的情况呢？
不会发生crash。

如果出现`dispatch_group_leave`比`dispatch_group_enter`多的情况呢？
会发生crash。

这里需要注意的是：
1. `dispatch_group_enter`和`dispatch_group_leave`是成对出现的。
    不管执行的顺序如何，必须保证成对出现，否则不会触发`dispatch_group_notify`。
2. `dispatch_group_notify`一般出现在最下边，有`dispatch_group_leave`就会触发，可以把3的代码放在最上边试一下。
3. `dispatch_group_enter`和`dispatch_group_leave`可以使用`dispatch_group_async`来代替，减少代码量以及没有成对出现可能带来的问题。
4. `dispatch_group_wait`设置超时时间，如果超时了任务还没有执行完成，则会直接触发`dispatch_group_notify`。

这些代码可以试一下哈，我们接下来看内部实现原理。


## dispatch_group_create

首先我们先看一下group的创建。

```
dispatch_group_t
dispatch_group_create(void)
{
	return _dispatch_group_create_with_count(0);
}
```

调用了`_dispatch_group_create_with_count`，参数是一个0

```
static inline dispatch_group_t
_dispatch_group_create_with_count(uint32_t n)
{
    // 1. 初始化dg，dispatch_group_s是一个结构体
	dispatch_group_t dg = _dispatch_object_alloc(DISPATCH_VTABLE(group),
			sizeof(struct dispatch_group_s));
	// 赋默认值
	dg->do_next = DISPATCH_OBJECT_LISTLESS;
	dg->do_targetq = _dispatch_get_default_queue(false);
	// n = 0，所以创建的时候永远不会执行这里。
	if (n) {
		os_atomic_store2o(dg, dg_bits,
				(uint32_t)-n * DISPATCH_GROUP_VALUE_INTERVAL, relaxed);
		os_atomic_store2o(dg, do_ref_cnt, 1, relaxed); // <rdar://22318411>
	}
	return dg;
}
```

创建group还是很好理解的，接下来看`dispatch_group_enter`

## dispatch_group_enter

```
void
dispatch_group_enter(dispatch_group_t dg)
{
	// The value is decremented on a 32bits wide atomic so that the carry
	// for the 0 -> -1 transition is not propagated to the upper 32bits.
	// dg->dg_bits的值从0 变成 -1
	uint32_t old_bits = os_atomic_sub_orig2o(dg, dg_bits,
			DISPATCH_GROUP_VALUE_INTERVAL, acquire);
	uint32_t old_value = old_bits & DISPATCH_GROUP_VALUE_MASK;
	// 等于0
	if (unlikely(old_value == 0)) {
		_dispatch_retain(dg); // <rdar://problem/22318411>
	}
	// 我们上面执行了多个enter之后，没有发生crash，但是这里也有解释
	// 超过一个最大值时也会发生crash
	if (unlikely(old_value == DISPATCH_GROUP_VALUE_MAX)) {
		DISPATCH_CLIENT_CRASH(old_bits,
				"Too many nested calls to dispatch_group_enter()");
	}
}
```

`os_atomic_sub_orig2o`这个函数跟上面信号量的函数有点类似，并且已经给了注释：dg->dg_bits的值从0 变成 -1。

## dispatch_group_leave

```
void
dispatch_group_leave(dispatch_group_t dg)
{
	// The value is incremented on a 64bits wide atomic so that the carry for
	// the -1 -> 0 transition increments the generation atomically.
	// dg_state从-1 -> 0 
	uint64_t new_state, old_state = os_atomic_add_orig2o(dg, dg_state,
			DISPATCH_GROUP_VALUE_INTERVAL, release);
	uint32_t old_value = (uint32_t)(old_state & DISPATCH_GROUP_VALUE_MASK);

	if (unlikely(old_value == DISPATCH_GROUP_VALUE_1)) {
		old_state += DISPATCH_GROUP_VALUE_INTERVAL;
		do {
			new_state = old_state;
			if ((old_state & DISPATCH_GROUP_VALUE_MASK) == 0) {
				new_state &= ~DISPATCH_GROUP_HAS_WAITERS;
				new_state &= ~DISPATCH_GROUP_HAS_NOTIFS;
			} else {
				// If the group was entered again since the atomic_add above,
				// we can't clear the waiters bit anymore as we don't know for
				// which generation the waiters are for
				new_state &= ~DISPATCH_GROUP_HAS_NOTIFS;
			}
			if (old_state == new_state) break;
		} while (unlikely(!os_atomic_cmpxchgv2o(dg, dg_state,
				old_state, new_state, &old_state, relaxed)));
		return _dispatch_group_wake(dg, old_state, true);
	}

  // old_value
	if (unlikely(old_value == 0)) {
		DISPATCH_CLIENT_CRASH((uintptr_t)old_value,
				"Unbalanced call to dispatch_group_leave()");
	}
}
```

我们一步一步的分析：

```
uint64_t new_state, old_state = os_atomic_add_orig2o(dg, dg_state,
		DISPATCH_GROUP_VALUE_INTERVAL, release);
```


这里有注释，写了解释内容，从-1变成了0，所以：
old_state = -1, new_state = 0 。


```
uint32_t old_value = (uint32_t)(old_state & DISPATCH_GROUP_VALUE_MASK);

#define DISPATCH_GROUP_VALUE_MASK       0x00000000fffffffcULL
```

old_value 是通过&运算得来的， -1 = 0xfffffffff，是一个全是1的二进制数，所以&运算之后old_value = DISPATCH_GROUP_VALUE_MASK。

如果再执行一次leave操作，那么old_state=0,然后经过&运算，old_value=0，也就是最后会发生crash。所以enter和leave一定要成对出现。

接下来就到重点了，当enter和leave达到平衡时，就会触发`_dispatch_group_wake`函数。

```
static void
_dispatch_group_wake(dispatch_group_t dg, uint64_t dg_state, bool needs_release)
{
	uint16_t refs = needs_release ? 1 : 0; // <rdar://problem/22318411>
    // &运算 
	if (dg_state & DISPATCH_GROUP_HAS_NOTIFS) {
		dispatch_continuation_t dc, next_dc, tail;

		// Snapshot before anything is notified/woken <rdar://problem/8554546>
		dc = os_mpsc_capture_snapshot(os_mpsc(dg, dg_notify), &tail);
		do {
			dispatch_queue_t dsn_queue = (dispatch_queue_t)dc->dc_data;
			next_dc = os_mpsc_pop_snapshot_head(dc, tail, do_next);
			// 执行这个操作，在第一章中有介绍，执行dx_push操作。直到block执行完成
			_dispatch_continuation_async(dsn_queue, dc,
					_dispatch_qos_from_pp(dc->dc_priority), dc->dc_flags);
			_dispatch_release(dsn_queue);
		} while ((dc = next_dc));

		refs++;
	}

	if (dg_state & DISPATCH_GROUP_HAS_WAITERS) {
		_dispatch_wake_by_address(&dg->dg_gen);
	}
   // 释放
	if (refs) _dispatch_release_n(dg, refs);
}
```

## dispatch_group_notify

接下来我们看一下`dispatch_group_notify`操作。

```
void
dispatch_group_notify(dispatch_group_t dg, dispatch_queue_t dq,
		dispatch_block_t db)
{
	dispatch_continuation_t dsn = _dispatch_continuation_alloc();
	_dispatch_continuation_init(dsn, dq, db, 0, DC_FLAG_CONSUME);
	_dispatch_group_notify(dg, dq, dsn);
}
```

`_dispatch_continuation_init` 内部是对属性赋值，保存dg(group)、dq(queue)、db(block)。在第一章有介绍。

`_dispatch_group_notify`的函数如下：


```
static inline void
_dispatch_group_notify(dispatch_group_t dg, dispatch_queue_t dq,
		dispatch_continuation_t dsn)
{
	uint64_t old_state, new_state;
	dispatch_continuation_t prev;

	dsn->dc_data = dq;
	_dispatch_retain(dq);

	prev = os_mpsc_push_update_tail(os_mpsc(dg, dg_notify), dsn, do_next);
	if (os_mpsc_push_was_empty(prev)) _dispatch_retain(dg);
	os_mpsc_push_update_prev(os_mpsc(dg, dg_notify), prev, dsn, do_next);
	if (os_mpsc_push_was_empty(prev)) {
		os_atomic_rmw_loop2o(dg, dg_state, old_state, new_state, release, {
			new_state = old_state | DISPATCH_GROUP_HAS_NOTIFS;
			// 重点哈~
			if ((uint32_t)old_state == 0) {
				os_atomic_rmw_loop_give_up({
					return _dispatch_group_wake(dg, new_state, false);
				});
			}
		});
	}
}
```

这里会有一系列的判断，当old_value=0的时候，执行wake操作。

## dispatch_group_async

```
void
dispatch_group_async(dispatch_group_t dg, dispatch_queue_t dq,
		dispatch_block_t db)
{
	dispatch_continuation_t dc = _dispatch_continuation_alloc();
	uintptr_t dc_flags = DC_FLAG_CONSUME | DC_FLAG_GROUP_ASYNC;
	dispatch_qos_t qos;
  // 任务包装器，存储block等信息
	qos = _dispatch_continuation_init(dc, dq, db, 0, dc_flags);
// 重点	_dispatch_continuation_group_async(dg, dq, dc, qos);
}
#endif
```


这里的重点代码是`_dispatch_continuation_group_async`.

```
static inline void
_dispatch_continuation_group_async(dispatch_group_t dg, dispatch_queue_t dq,
		dispatch_continuation_t dc, dispatch_qos_t qos)
{
	dispatch_group_enter(dg);
	dc->dc_data = dg;
	_dispatch_continuation_async(dq, dc, qos, dc->dc_flags);
}
```

看到了吧，其内部有自动执行`dispatch_group_enter`操作。但是什么时候执行的leave呢？还记得上一章中介绍`_dispatch_continuation_invoke_inline`的时候吗？里头有一局代码是关于group的操作，我还专门写了注释。

`_dispatch_continuation_async`是执行dx_push操作。

`_dispatch_continuation_invoke_inline`在group的情况会值执行group相关的逻辑：

```
static inline void
_dispatch_continuation_with_group_invoke(dispatch_continuation_t dc)
{
	struct dispatch_object_s *dou = dc->dc_data;
	unsigned long type = dx_type(dou);
	if (type == DISPATCH_GROUP_TYPE) {
		_dispatch_client_callout(dc->dc_ctxt, dc->dc_func);
		_dispatch_trace_item_complete(dc);
		dispatch_group_leave((dispatch_group_t)dou);
	} else {
		DISPATCH_INTERNAL_CRASH(dx_type(dou), "Unexpected object type");
	}
}
```

嗯哼~这里也会执行`dispatch_group_leave`操作，这也就是dispatch_group_async可以代替enter和leave的原因。
下面的代码可以很好的解释`dispatch_group_async`的原理

```
dispatch_group_async(dispatch_group_t group, dispatch_queue_t queue, dispatch_block_t block)
{
	dispatch_retain(group);
	dispatch_group_enter(group);
	dispatch_async(queue, ^{
		block();
		dispatch_group_leave(group);
		dispatch_release(group);
	});
}
```

# dispatch_source

`dispatch_source`是一个更为底层，直接与内核交互的东西，所以它执行起来会更快，效率更高。所以type类型是time的时候，准确率是最高的。

```
NSUInteger totalComplete = 0
// 创建DISPATCH_SOURCE_TYPE_DATA_ADD类型的source，再主线程执行
dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_DATA_ADD, 0, 0, dispatch_get_main_queue());

// 设置block回调
dispatch_source_set_event_handler(self.source, ^{
    // 获取数据
    NSUInteger value = dispatch_source_get_data(self.source);
    totalComplete += value;
    NSLog(@"进度: %.2f", totalComplete/100.0);
});

// 开始执行
dispatch_resume(source);

// 异步执行for
for (int i= 0; i<100; i++) {
    dispatch_async(self.queue, ^{
        sleep(1);
        // merge数据，每次+1，每次merge就会触发dispatch_source_set_event_handler
        dispatch_source_merge_data(self.source, 1);
    });
}
    
    
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(100 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    // 停止
    dispatch_suspend(_source);
});
```

上面内容就是source相关的。不是很常用，了解一下。


# 总结

1. dispatch_semaphore信号量
    1. `dispatch_semaphore_signal`与`dispatch_semaphore_wait`成对出现。
    2. `dispatch_semaphore_signal`是+1操作
    3. `dispatch_semaphore_wait`是-1操作
2. dispatch_group调度组
    1. enter和leave是成对出现的，否则可能发生crash
    2. `dispatch_group_notify`一般放在最下边执行。
    3. `dispatch_group_async`可以替代enter和leave两个操作。
    4. `dispatch_group_wait`超时操作。
3. source，每次执行merge操作就会执行block。


以上就是GCD的相关内容了，写了3章，有不对的地方欢迎指正。