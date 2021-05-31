---
title: runloop
date: 2021-05-30 13:34:10
tags:
    - Objective-C,
    - iOS
categories:
    - OC原理
---

# Runloop

## 简介

RunLoop是事件接收和分发机制的一个实现，是线程相关的基础框架的一部分，一个RunLoop就是一个事件处理的循环，用来不停的调度工作以及处理输入事件。

RunLoop本质是一个 do-while循环，没事做就休息，来活了就干活。与普通的while循环是有区别的，普通的while循环会导致CPU进入忙等待状态，即一直消耗cpu，而RunLoop则不会，RunLoop是一种闲等待，即RunLoop具备休眠功能。

## RunLoop的作用

* 保持程序的持续运行
* 处理App中的各种事件（触摸、定时器、performSelector）
* 节省cpu资源，提供程序的性能，该做事就做事，该休息就休息

# 源码分析

[源码下载](https://opensource.apple.com/tarballs/CF/)

## runloop与线程

通常情况下获取runloop的两种方式：

```
// 主运行循环
CFRunLoopRef mainRunloop = CFRunLoopGetMain();
// 当前运行循环
CFRunLoopRef currentRunloop = CFRunLoopGetCurrent();
```

接下来看一下源码：

```
CFRunLoopRef CFRunLoopGetMain(void) {
    CHECK_FOR_FORK();
    // 这是一个静态变量
    static CFRunLoopRef __main = NULL; // no retain needed
    // 没有获取到，则通过_CFRunLoopGet0函数去获取，参数是主线程
    if (!__main) __main = _CFRunLoopGet0(pthread_main_thread_np()); // no CAS needed
    return __main;
}
```

查看一下`_CFRunLoopGet0`函数：

```
CF_EXPORT CFRunLoopRef _CFRunLoopGet0(pthread_t t) {
    // 如参数t不存在，则默认为主线程
    if (pthread_equal(t, kNilPthreadT)) {
        t = pthread_main_thread_np();
    }
    __CFSpinLock(&loopsLock);
    if (!__CFRunLoops) {
        __CFSpinUnlock(&loopsLock);
        
        // 创建一个字典
        CFMutableDictionaryRef dict = CFDictionaryCreateMutable(kCFAllocatorSystemDefault, 0, NULL, &kCFTypeDictionaryValueCallBacks);
        // 创建mainLoop
        CFRunLoopRef mainLoop = __CFRunLoopCreate(pthread_main_thread_np());
        
        // dict : key value
        // 把main_thread和mainloop通过key-value的形式绑定
        CFDictionarySetValue(dict, pthreadPointer(pthread_main_thread_np()), mainLoop);
        
        if (!OSAtomicCompareAndSwapPtrBarrier(NULL, dict, (void * volatile *)&__CFRunLoops)) {
            CFRelease(dict);
        }
        
        CFRelease(mainLoop);
        __CFSpinLock(&loopsLock);
    }
    
    // 从字典中通过线程获取run loop
    CFRunLoopRef loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
    __CFSpinUnlock(&loopsLock);
    if (!loop) {
        // 没有则创建
        CFRunLoopRef newLoop = __CFRunLoopCreate(t);
        __CFSpinLock(&loopsLock);
        loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
        if (!loop) {
            // 没有loop也要存，存的是新创建的。
            CFDictionarySetValue(__CFRunLoops, pthreadPointer(t), newLoop);
            loop = newLoop;
        }
        // don't release run loops inside the loopsLock, because CFRunLoopDeallocate may end up taking it
        __CFSpinUnlock(&loopsLock);
        CFRelease(newLoop);
    }
    if (pthread_equal(t, pthread_self())) {
        _CFSetTSD(__CFTSDKeyRunLoop, (void *)loop, NULL);
        if (0 == _CFGetTSD(__CFTSDKeyRunLoopCntr)) {
            _CFSetTSD(__CFTSDKeyRunLoopCntr, (void *)(PTHREAD_DESTRUCTOR_ITERATIONS-1), (void (*)(void *))__CFFinalizeRunLoop);
        }
    }
    return loop;
}
```

上面的代码可以看出，runloo只有两种类型，一种主线程的mainloop，还有就是其它runloop。

## runloop的创建

接下来看runloop是怎么创建的：

```
static CFRunLoopRef __CFRunLoopCreate(pthread_t t) {
    CFRunLoopRef loop = NULL;
    CFRunLoopModeRef rlm;
    uint32_t size = sizeof(struct __CFRunLoop) - sizeof(CFRuntimeBase);
    loop = (CFRunLoopRef)_CFRuntimeCreateInstance(kCFAllocatorSystemDefault, __kCFRunLoopTypeID, size, NULL);
    // 如果loop为空，则直接返回NULL
    if (NULL == loop) {
        return NULL;
    }
    // runloop属性赋值
    (void)__CFRunLoopPushPerRunData(loop);
    __CFRunLoopLockInit(&loop->_lock);
    loop->_wakeUpPort = __CFPortAllocate();
    if (CFPORT_NULL == loop->_wakeUpPort) HALT;
    __CFRunLoopSetIgnoreWakeUps(loop);
    loop->_commonModes = CFSetCreateMutable(kCFAllocatorSystemDefault, 0, &kCFTypeSetCallBacks);
    CFSetAddValue(loop->_commonModes, kCFRunLoopDefaultMode);
    loop->_commonModeItems = NULL;
    loop->_currentMode = NULL;
    loop->_modes = CFSetCreateMutable(kCFAllocatorSystemDefault, 0, &kCFTypeSetCallBacks);
    loop->_blocks_head = NULL;
    loop->_blocks_tail = NULL;
    loop->_counterpart = NULL;
    loop->_pthread = t;
#if DEPLOYMENT_TARGET_WINDOWS
    loop->_winthread = GetCurrentThreadId();
#else
    loop->_winthread = 0;
#endif
    rlm = __CFRunLoopFindMode(loop, kCFRunLoopDefaultMode, true);
    if (NULL != rlm) __CFRunLoopModeUnlock(rlm);
    return loop;
}
```

里面又有了一个CFRunLoopRef,盲猜应该是结构体。

```
typedef struct __CFRunLoop * CFRunLoopRef;

struct __CFRunLoop {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;            /* locked for accessing mode list */
    __CFPort _wakeUpPort;            // used for CFRunLoopWakeUp
    Boolean _unused;
    volatile _per_run_data *_perRunData;              // reset for runs of the run loop
    pthread_t _pthread;
    uint32_t _winthread;
    CFMutableSetRef _commonModes;
    CFMutableSetRef _commonModeItems;
    CFRunLoopModeRef _currentMode;
    CFMutableSetRef _modes;
    struct _block_item *_blocks_head;
    struct _block_item *_blocks_tail;
    CFTypeRef _counterpart;
};
```

从定义中可以得出，一个RunLoop有多个Mode，意味着一个RunLoop需要处理多个事务，即一个Mode对应多个Item，而一个item中，包含了timer、source、observer，如图：

![](runloop_1.jpg)

### mode类型

其中mode在苹果文档中提及的有五个，而在iOS中公开暴露出来的只有 `NSDefaultRunLoopMode`和`NSRunLoopCommonModes`。

`NSRunLoopCommonModes` 实际上是一个 Mode 的集合，默认包括 `NSDefaultRunLoopMode` 和 `NSEventTrackingRunLoopMode`。

* NSDefaultRunLoopMode：默认的mode，正常情况下都是在这个mode
* NSConnectionReplyMode
* NSModalPanelRunLoopMode
* NSEventTrackingRunLoopMode：使用这个Mode去跟踪来自用户交互的事件（比如UITableView上下滑动）
* NSRunLoopCommonModes：伪模式，灵活性更好

### source

* Source0 表示 非系统事件，即用户自定义的事件
* Source1 表示系统事件，主要负责底层的通讯，具备唤醒能力

### Observer

```
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    //进入RunLoop
    kCFRunLoopEntry = (1UL << 0),
    //即将处理Timers
    kCFRunLoopBeforeTimers = (1UL << 1),
    //即将处理Source
    kCFRunLoopBeforeSources = (1UL << 2),
    //即将进入休眠
    kCFRunLoopBeforeWaiting = (1UL << 5),
    //被唤醒
    kCFRunLoopAfterWaiting = (1UL << 6),
    //退出RunLoop
    kCFRunLoopExit = (1UL << 7),
    kCFRunLoopAllActivities = 0x0FFFFFFFU
};
```

### mode对应的items

* block：__CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__
* timer：__CFRUNLOOP_IS_CALLING_OUT_TO_A_TIMER_CALLBACK_FUNCTION__
* source0： __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__
* source1： __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE1_PERFORM_FUNCTION__
* 主队列：__CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__
*observer： __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__

### 以Timer为例

在子线程创建的timer是没有办法一直执行的，而想让它继续执行，则需要添加到runloop中，并且run才行。

```
self.isStopping = NO;
NSThread *thread = [[NSThread alloc] initWithBlock:^{

    // thread.name = nil 因为这个变量只是捕捉
    // LGThread *thread = nil
    // thread = 初始化 捕捉一个nil进来
    NSLog(@"%@---%@",[NSThread currentThread],[[NSThread currentThread] name]);
    NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:1 repeats:YES block:^(NSTimer * _Nonnull timer) {
        NSLog(@"~~hello word");            // 退出线程--结果runloop也停止了
        if (self.isStopping) {
            [NSThread exit];
        }
    }];
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
    [[NSRunLoop currentRunLoop] run];
}];

thread.name = @"lgcode.com";
[thread start];
```

我们看一下addTimer是怎么操作的。

```
oid CFRunLoopAddTimer(CFRunLoopRef rl, CFRunLoopTimerRef rlt, CFStringRef modeName) {
    CHECK_FOR_FORK();
    if (__CFRunLoopIsDeallocating(rl)) return;
    if (!__CFIsValid(rlt) || (NULL != rlt->_runLoop && rlt->_runLoop != rl)) return;
    __CFRunLoopLock(rl);
    
    // 重点 : kCFRunLoopCommonModes
    if (modeName == kCFRunLoopCommonModes) {
        //如果是kCFRunLoopCommonModes 类型
        CFSetRef set = rl->_commonModes ? CFSetCreateCopy(kCFAllocatorSystemDefault, rl->_commonModes) : NULL;
        
        if (NULL == rl->_commonModeItems) {
            // modeItems是空，则创建一个defalut
            rl->_commonModeItems = CFSetCreateMutable(kCFAllocatorSystemDefault, 0, &kCFTypeSetCallBacks);
        }
        //runloop与mode 是一对多的， mode与item也是一对多的
        CFSetAddValue(rl->_commonModeItems, rlt);
        if (NULL != set) {
            CFTypeRef context[2] = {rl, rlt};
            /* add new item to all common-modes */
            //执行
            CFSetApplyFunction(set, (__CFRunLoopAddItemToCommonModes), (void *)context);
            CFRelease(set);
        }
    } else {
        //如果是非commonMode类型
        //查找runloop的模型
        CFRunLoopModeRef rlm = __CFRunLoopFindMode(rl, modeName, true);
        if (NULL != rlm) {
            if (NULL == rlm->_timers) {
                CFArrayCallBacks cb = kCFTypeArrayCallBacks;
                cb.equal = NULL;
                rlm->_timers = CFArrayCreateMutable(kCFAllocatorSystemDefault, 0, &cb);
            }
        }
        //判断mode是否匹配
        if (NULL != rlm && !CFSetContainsValue(rlt->_rlModes, rlm->_name)) {
            __CFRunLoopTimerLock(rlt);
            if (NULL == rlt->_runLoop) {
                rlt->_runLoop = rl;
            } else if (rl != rlt->_runLoop) {
                __CFRunLoopTimerUnlock(rlt);
                __CFRunLoopModeUnlock(rlm);
                __CFRunLoopUnlock(rl);
                return;
            }
            // 如果匹配，则将runloop加进去，而runloop的执行依赖于  [runloop run]
            CFSetAddValue(rlt->_rlModes, rlm->_name);
            __CFRunLoopTimerUnlock(rlt);
            __CFRunLoopTimerFireTSRLock();
            __CFRepositionTimerInMode(rlm, rlt, false);
            __CFRunLoopTimerFireTSRUnlock();
            if (!_CFExecutableLinkedOnOrAfter(CFSystemVersionLion)) {
                // Normally we don't do this on behalf of clients, but for
                // backwards compatibility due to the change in timer handling...
                if (rl != CFRunLoopGetCurrent()) CFRunLoopWakeUp(rl);
            }
        }
        if (NULL != rlm) {
            __CFRunLoopModeUnlock(rlm);
        }
    }
   
    __CFRunLoopUnlock(rl);
}
```

主要目的就是把timer添加到对应的mode中。mode 和 item是一对多的关系，timer是item的一种。

### __CFRunLoopRun

接下来上源码：

```
/* rl, rlm are locked on entrance and exit */
static int32_t __CFRunLoopRun(CFRunLoopRef rl, CFRunLoopModeRef rlm, CFTimeInterval seconds, Boolean stopAfterHandle, CFRunLoopModeRef previousMode) {
    
    ...
    do {
        ...
        
        __CFRunLoopUnsetIgnoreWakeUps(rl);
        
        if (rlm->_observerMask & kCFRunLoopBeforeTimers) __CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeTimers);
        if (rlm->_observerMask & kCFRunLoopBeforeSources) __CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeSources);
        
        __CFRunLoopDoBlocks(rl, rlm);
        
        Boolean sourceHandledThisLoop = __CFRunLoopDoSources0(rl, rlm, stopAfterHandle);
        if (sourceHandledThisLoop) {
            __CFRunLoopDoBlocks(rl, rlm);
        }
        ...
        
        //如果是timer
        else if (modeQueuePort != MACH_PORT_NULL && livePort == modeQueuePort) {
            CFRUNLOOP_WAKEUP_FOR_TIMER();
            if (!__CFRunLoopDoTimers(rl, rlm, mach_absolute_time())) {
                // Re-arm the next timer, because we apparently fired early
                __CFArmNextTimerInMode(rlm, rl);
            }
        }
        
        ...
        
        //如果是source1
        CFRunLoopSourceRef rls = __CFRunLoopModeFindSourceForMachPort(rl, rlm, livePort);
        if (rls) {
#if DEPLOYMENT_TARGET_MACOSX || DEPLOYMENT_TARGET_EMBEDDED || DEPLOYMENT_TARGET_EMBEDDED_MINI
            mach_msg_header_t *reply = NULL;
            sourceHandledThisLoop = __CFRunLoopDoSource1(rl, rlm, rls, msg, msg->msgh_size, &reply) || sourceHandledThisLoop;
            if (NULL != reply) {
                (void)mach_msg(reply, MACH_SEND_MSG, reply->msgh_size, 0, MACH_PORT_NULL, 0, MACH_PORT_NULL);
                CFAllocatorDeallocate(kCFAllocatorSystemDefault, reply);
            }
#elif DEPLOYMENT_TARGET_WINDOWS
            sourceHandledThisLoop = __CFRunLoopDoSource1(rl, rlm, rls) || sourceHandledThisLoop;
#endif
        }
        ...
    
    }while (0 == retVal);
    
    ...
}
```

__CFRunLoopDoTimers源码，主要是通过for循环，对单个timer进行处理。


```
static Boolean __CFRunLoopDoTimers(CFRunLoopRef rl, CFRunLoopModeRef rlm, uint64_t limitTSR) {    /* DOES CALLOUT */
    ...
    //循环遍历，做下层单个timer的执行
    for (CFIndex idx = 0, cnt = timers ? CFArrayGetCount(timers) : 0; idx < cnt; idx++) {
        CFRunLoopTimerRef rlt = (CFRunLoopTimerRef)CFArrayGetValueAtIndex(timers, idx);
        // 执行timer
        Boolean did = __CFRunLoopDoTimer(rl, rlm, rlt);
        timerHandled = timerHandled || did;
    }
    ...
}
```

```
// mode and rl are locked on entry and exit
static Boolean __CFRunLoopDoTimer(CFRunLoopRef rl, CFRunLoopModeRef rlm, CFRunLoopTimerRef rlt) {    /* DOES CALLOUT */
    Boolean timerHandled = false;
    uint64_t oldFireTSR = 0;
    
    /* Fire a timer */
    CFRetain(rlt);
    __CFRunLoopTimerLock(rlt);
    if (__CFIsValid(rlt) && rlt->_fireTSR <= mach_absolute_time() && !__CFRunLoopTimerIsFiring(rlt) && rlt->_runLoop == rl) {
    __CFRunLoopTimerUnlock(rlt);
        __CFRunLoopTimerFireTSRLock();
        oldFireTSR = rlt->_fireTSR;
        __CFRunLoopTimerFireTSRUnlock();
        
        __CFArmNextTimerInMode(rlm, rl);
        
        __CFRunLoopModeUnlock(rlm);
        __CFRunLoopUnlock(rl);
        // 执行timer
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_TIMER_CALLBACK_FUNCTION__(rlt->_callout, rlt, context_info);
        CHECK_FOR_FORK();
        if (doInvalidate) {
            CFRunLoopTimerInvalidate(rlt);      /* DOES CALLOUT */
        }
        if (context_release) {
            context_release(context_info);
        }
        __CFRunLoopLock(rl);
        __CFRunLoopModeLock(rlm);
        __CFRunLoopTimerLock(rlt);
        timerHandled = true;
        __CFRunLoopTimerUnsetFiring(rlt);
    }
}
```

在timer执行的位置打上断点，使用lldb -> bt命令查看调用栈：

![](runloop_2.jpg)

### timer的调用顺序

1. 自定义的timer，设置Mode，并将其加入RunLoop中
2. 在RunLoop的run方法执行时，会调用__CFRunLoopDoTimers执行所有timer
3. 在__CFRunLoopDoTimers方法中，会通过for循环执行单个timer的操作
4. 在__CFRunLoopDoTimer方法中，timer执行完毕后，会执行对应的timer回调函数

是针对timer的执行分析，对于observer、block、source0、source1，其执行原理与timer是类似的

![](runloop_3.jpg)

# runloop底层原理

```
void CFRunLoopRun(void) {    /* DOES CALLOUT */
    int32_t result;
    do {
        // 1.0e10 : 科学计数 1*10^10，很大的值
        result = CFRunLoopRunSpecific(CFRunLoopGetCurrent(), kCFRunLoopDefaultMode, 1.0e10, false);
        CHECK_FOR_FORK();
    } while (kCFRunLoopRunStopped != result && kCFRunLoopRunFinished != result);
}
```

runloop就是一个do-while循环。当stop或者执行完成之后，则退出循环。

看一下`CFRunLoopRunSpecific`的内部：


```
SInt32 CFRunLoopRunSpecific(CFRunLoopRef rl, CFStringRef modeName, CFTimeInterval seconds, Boolean returnAfterSourceHandled) {     /* DOES CALLOUT */
    CHECK_FOR_FORK();
    if (__CFRunLoopIsDeallocating(rl)) return kCFRunLoopRunFinished;
    __CFRunLoopLock(rl);
    
    //首先根据modeName找到对应mode
    CFRunLoopModeRef currentMode = __CFRunLoopFindMode(rl, modeName, false);
    
    // 通知 Observers: RunLoop 即将进入 loop。
    __CFRunLoopDoObservers(rl, currentMode, kCFRunLoopEntry);
    
    // 内部函数，进入loop，seconds是一个很大的值
    result = __CFRunLoopRun(rl, currentMode, seconds, returnAfterSourceHandled, previousMode);
    
    // 通知 Observers: RunLoop 即将退出。
    __CFRunLoopDoObservers(rl, currentMode, kCFRunLoopExit);
    
    return result;
}
```

接下来又回到`__CFRunLoopRun`的代码，上面提到的逻辑只是针对timer的，这里详细的说明一下，使用伪代码：

```
//核心函数
/* rl, rlm are locked on entrance and exit */
static int32_t __CFRunLoopRun(CFRunLoopRef rl, CFRunLoopModeRef rlm, CFTimeInterval seconds, Boolean stopAfterHandle, CFRunLoopModeRef previousMode){
    
    //通过GCD开启一个定时器，然后开始跑圈
    dispatch_source_t timeout_timer = NULL;
    ...
    dispatch_resume(timeout_timer);
    
    int32_t retVal = 0;
    
    //处理事务,即处理items
    do {
        
        // 通知 Observers: 即将处理timer事件
        __CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeTimers);
        
        // 通知 Observers: 即将处理Source事件
        __CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeSources)
        
        // 处理Blocks
        __CFRunLoopDoBlocks(rl, rlm);
        
        // 处理sources0
        Boolean sourceHandledThisLoop = __CFRunLoopDoSources0(rl, rlm, stopAfterHandle);
        
        // 处理sources0返回为YES
        if (sourceHandledThisLoop) {
            // 处理Blocks
            __CFRunLoopDoBlocks(rl, rlm);
        }
        
        // 判断有无端口消息(Source1)
        if (__CFRunLoopWaitForMultipleObjects(NULL, &dispatchPort, 0, 0, &livePort, NULL)) {
            // 处理消息
            goto handle_msg;
        }
        
        
        // 通知 Observers: 即将进入休眠
        __CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeWaiting);
        __CFRunLoopSetSleeping(rl);
        
        // 等待被唤醒
        __CFRunLoopServiceMachPort(waitSet, &msg, sizeof(msg_buffer), &livePort, poll ? 0 : TIMEOUT_INFINITY);
        
        // user callouts now OK again
        __CFRunLoopUnsetSleeping(rl);
        
        // 通知 Observers: 被唤醒，结束休眠
        __CFRunLoopDoObservers(rl, rlm, kCFRunLoopAfterWaiting);
        
        
    handle_msg:
        if (被timer唤醒) {
            // 处理Timers
            __CFRunLoopDoTimers(rl, rlm, mach_absolute_time())；
        }else if (被GCD唤醒){
            // 处理gcd
            __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__(msg);
        }else if (被source1唤醒){
            // 被Source1唤醒，处理Source1
            __CFRunLoopDoSource1(rl, rlm, rls, msg, msg->msgh_size, &reply)
        }
        
        // 处理block
        __CFRunLoopDoBlocks(rl, rlm);
        
        if (sourceHandledThisLoop && stopAfterHandle) {
            retVal = kCFRunLoopRunHandledSource;//处理源
        } else if (timeout_context->termTSR < mach_absolute_time()) {
            retVal = kCFRunLoopRunTimedOut;//超时
        } else if (__CFRunLoopIsStopped(rl)) {
            __CFRunLoopUnsetStopped(rl);
            retVal = kCFRunLoopRunStopped;//停止
        } else if (rlm->_stopped) {
            rlm->_stopped = false;
            retVal = kCFRunLoopRunStopped;//停止
        } else if (__CFRunLoopModeIsEmpty(rl, rlm, previousMode)) {
            retVal = kCFRunLoopRunFinished;//结束
        }
    }while (0 == retVal);
    
    return retVal;
}
```

整理一下runloop的整体流程如下：

![](runloop_4.jpg)

# 总结



# 引用

[源码下载](https://opensource.apple.com/tarballs/CF/)
