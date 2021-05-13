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

```

这个方法什么时候有用到，没有看错，在`_dispatch_root_queues_init_once`
