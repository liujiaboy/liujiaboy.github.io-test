---
title: GCD-2
date: 2021-05-12 21:32:39
tags:
    - Objective-C,
    - iOS
categories:
    - OC原理
---

# dispatch_async异步函数的调用

我们继续上一章的内容继续研究一下block的调用，在`NSlog`这里打一个断点，看看调用的函数栈。

```
dispatch_async(conque, ^{
    NSLog(@"12334");
});
```

我们运行代码，通过bt命令看一下调用栈：

```
(lldb) bt
* thread #8, queue = 'conque', stop reason = breakpoint 2.1
  * frame #0: 0x0000000103abd0d7 __29-[ViewController viewDidLoad]_block_invoke(.block_descriptor=0x0000000103ac00e8) at ViewController.m:48:9
    frame #1: 0x0000000103d2e7ec libdispatch.dylib`_dispatch_call_block_and_release + 12
    frame #2: 0x0000000103d2f9c8 libdispatch.dylib`_dispatch_client_callout + 8
    frame #3: 0x0000000103d32316 libdispatch.dylib`_dispatch_continuation_pop + 557
    frame #4: 0x0000000103d3171c libdispatch.dylib`_dispatch_async_redirect_invoke + 779
    frame #5: 0x0000000103d41508 libdispatch.dylib`_dispatch_root_queue_drain + 351
    frame #6: 0x0000000103d41e6d libdispatch.dylib`_dispatch_worker_thread2 + 135
    frame #7: 0x00007fff60c8e453 libsystem_pthread.dylib`_pthread_wqthread + 244
    frame #8: 0x00007fff60c8d467 libsystem_pthread.dylib`start_wqthread + 15
```

从下往上看哈~函数的调用竟然是通过与pthread交互之后发生的。然后到了`_dispatch_worker_thread2`：

```
static void
_dispatch_worker_thread2(pthread_priority_t pp)
{
	bool overcommit = pp & _PTHREAD_PRIORITY_OVERCOMMIT_FLAG;
	dispatch_queue_global_t dq;

	pp &= _PTHREAD_PRIORITY_OVERCOMMIT_FLAG | ~_PTHREAD_PRIORITY_FLAGS_MASK;
	_dispatch_thread_setspecific(dispatch_priority_key, (void *)(uintptr_t)pp);
	dq = _dispatch_get_root_queue(_dispatch_qos_from_pp(pp), overcommit);

	_dispatch_introspection_thread_add();
	_dispatch_trace_runtime_event(worker_unpark, dq, 0);

	int pending = os_atomic_dec2o(dq, dgq_pending, relaxed);
	dispatch_assert(pending >= 0);
	_dispatch_root_queue_drain(dq, dq->dq_priority,
			DISPATCH_INVOKE_WORKER_DRAIN | DISPATCH_INVOKE_REDIRECTING_DRAIN);
	_dispatch_voucher_debug("root queue clear", NULL);
	_dispatch_reset_voucher(NULL, DISPATCH_THREAD_PARK);
	_dispatch_trace_runtime_event(worker_park, NULL, 0);
}
```

根据函数调用栈，来到`_dispatch_root_queue_drain`这个函数。函数内容做了删减。

```
static void
_dispatch_root_queue_drain(dispatch_queue_global_t dq,
		dispatch_priority_t pri, dispatch_invoke_flags_t flags)
{
...
  // 设置当前queue
	_dispatch_queue_set_current(dq);
	_dispatch_init_basepri(pri);
	_dispatch_adopt_wlh_anon();

	struct dispatch_object_s *item;
	bool reset = false;
	dispatch_invoke_context_s dic = { };
#if DISPATCH_COCOA_COMPAT
	_dispatch_last_resort_autorelease_pool_push(&dic);
#endif // DISPATCH_COCOA_COMPAT
	_dispatch_queue_drain_init_narrowing_check_deadline(&dic, pri);
	_dispatch_perfmon_start();
	while (likely(item = _dispatch_root_queue_drain_one(dq))) {
		if (reset) _dispatch_wqthread_override_reset();
		// 函数重点
		_dispatch_continuation_pop_inline(item, &dic, flags, dq);
		reset = _dispatch_reset_basepri_override();
		if (unlikely(_dispatch_queue_drain_should_narrow(&dic))) {
			break;
		}
	}

	...
	_dispatch_reset_wlh();
	_dispatch_clear_basepri();
	// 设置当前queue为NULL
	_dispatch_queue_set_current(NULL);
}
```

这个函数内一开始需要将当前队列调回来，然后执行block中的内容，完成之后，在把对列置空。block内部怎么调用，就在`_dispatch_continuation_pop_inline`里头。

```
static inline void
_dispatch_continuation_pop_inline(dispatch_object_t dou,
		dispatch_invoke_context_t dic, dispatch_invoke_flags_t flags,
		dispatch_queue_class_t dqu)
{
	dispatch_pthread_root_queue_observer_hooks_t observer_hooks =
			_dispatch_get_pthread_root_queue_observer_hooks();
	if (observer_hooks) observer_hooks->queue_will_execute(dqu._dq);
	flags &= _DISPATCH_INVOKE_PROPAGATE_MASK;
	if (_dispatch_object_has_vtable(dou)) {
		dx_invoke(dou._dq, dic, flags);
	} else {
		_dispatch_continuation_invoke_inline(dou, flags, dqu);
	}
	if (observer_hooks) observer_hooks->queue_did_execute(dqu._dq);
}
```

我们猜测有可能执行的是`_dispatch_continuation_invoke_inline`，因为其他的看着也不咋像那回事。

```
static inline void
_dispatch_continuation_invoke_inline(dispatch_object_t dou,
		dispatch_invoke_flags_t flags, dispatch_queue_class_t dqu)
{
	dispatch_continuation_t dc = dou._dc, dc1;
	dispatch_invoke_with_autoreleasepool(flags, {
		uintptr_t dc_flags = dc->dc_flags;
		_dispatch_continuation_voucher_adopt(dc, dc_flags);
		if (!(dc_flags & DC_FLAG_NO_INTROSPECTION)) {
			_dispatch_trace_item_pop(dqu, dou);
		}
		if (dc_flags & DC_FLAG_CONSUME) {
			dc1 = _dispatch_continuation_free_cacheonly(dc);
		} else {
			dc1 = NULL;
		}
		if (unlikely(dc_flags & DC_FLAG_GROUP_ASYNC)) {
			_dispatch_continuation_with_group_invoke(dc);
		} else {
			_dispatch_client_callout(dc->dc_ctxt, dc->dc_func);
			_dispatch_trace_item_complete(dc);
		}
		if (unlikely(dc1)) {
			_dispatch_continuation_free_to_cache_limit(dc1);
		}
	});
	_dispatch_perfmon_workitem_inc();
}
```

`dispatch_invoke_with_autoreleasepool`这里有一个`autoreleasepool`，源码内部对自动释放池的操作还是很严谨的。

从始至终，一直执行的都是likely的语句，所以这里执行`_dispatch_client_callout`。

```
void
_dispatch_client_callout(void *ctxt, dispatch_function_t f)
{
	_dispatch_get_tsd_base();
	void *u = _dispatch_get_unwind_tsd();
	// 执行这里
	if (likely(!u)) return f(ctxt);
	_dispatch_set_unwind_tsd(NULL);
	f(ctxt);
	_dispatch_free_unwind_tsd();
	_dispatch_set_unwind_tsd(u);
}
```

看到了木有啊，f还记得是啥吗？回到上一章的这个`_dispatch_continuation_init`的函数中，有解释哦，f就等于`_dispatch_call_block_and_release`。

我们再回过头看看打印的函数调用栈，最后执行的不就是`_dispatch_call_block_and_release`吗？前面的dispatch_asyn内部实现对block进行保存，这里进行调用。

到此为止整个异步函数的调用就结束了。


结合上一章的`dispatch_async`的内容，我们可以通过汇编添加`symbolic breakpoint`进行判断，我们所分析的函数执行步奏是否正确，在不知道执行流程的情况下，添加断点可以让我们比较清楚的知道其内部是怎么执行的。这里就不去操作了哈~











