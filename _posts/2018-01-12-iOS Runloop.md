---
layout:         post
title:          iOS Runloop分析
card-image: http://ouo0w16s7.bkt.clouddn.com/iOS%20runloop.png?e=1515742507&token=X-k_MFo98a0VybdWmejDMbKtt0VLy2zl6vAXUB4r:odryosovHNx1tW7U4S0UaeXS4Cg
subtitle:       
location:  北京 中国   
date:           2018-01-12 14:27:00
categories: iOS
tags:  iOS Runloop
excerpt:  最近面试，结合网络博客复习了下 Runtime 相关知识，做一下总结。本文简单介绍了什么是运行时，以及运行时在iOS开发过程中的几个常用场景。
---
## 1. 什么是 Runloop 
runloop 实际上是一个管理需要处理的事件和消息，并做出响应处理的循环。

## 2.Runloop 的作用
### 2.1	保证程序的持续运行

程序启动，则开启一个主线程。主线程一开启，则会跑起一个对应的 Runloop。Runloop 保证线程不会被销毁，也就宝成程序的持续运行。

### 2.2 处理 App 的各种事件
程序运行期间，App 所接收到的触摸事件、定时器事件、Selector 事件等，均需要在 Runloop 内处理。

### 2.3 节省 CPU 资源，提高程序性能
Runloop 存在休眠机制，在没有事件需要处理时，Runloop 会进入休眠机制，节省 CPU 资源，直至新的消息到来才会立刻唤醒。

## 3.Runloop 与线程的关系
* 每条线程都有唯一的一个与之对应的 RunLoop 对象
* 主线程的 RunLoop 已经自动创建好了，子线程的 RunLoop 需要主动创建
* RunLoop 在第一次获取时创建，在线程结束时销毁

### 3.1 Runloop 的创建

线程和 RunLoop 之间是一一对应的，其关系是保存在一个 Dictionary 里。所以我们创建子线程 RunLoop 时，只需在子线程中获取当前线程的 RunLoop 对象即可[NSRunLoop currentRunLoop]；如果不获取，那子线程就不会创建与之相关联的 RunLoop，并且只能在一个线程的内部获取其 RunLoop。[NSRunLoop currentRunLoop] 方法调用时，会先看一下字典里有没有存子线程相对用的 RunLoop，如果有则直接返回 RunLoop，如果没有则会创建一个，并将与之对应的子线程存入字典中。

#### 3.1.1 创建与主线程相关联的 RunLoop 

```objc

// 创建字典
CFMutableDictionaryRef dict = CFDictionaryCreateMutable(kCFAllocatorSystemDefault, 0, NULL, &kCFTypeDictionaryValueCallBacks);
        
// 创建主线程 根据传入的主线程创建主线程对应的RunLoop
CFRunLoopRef mainLoop = __CFRunLoopCreate(pthread_main_thread_np());
        
// 保存主线程 将主线程-key和RunLoop-Value保存到字典中
CFDictionarySetValue(dict, pthreadPointer(pthread_main_thread_np()), mainLoop);
```

#### 3.1.2 创建与子线程相关联的 RunLoop

```objc
// 从字典中获取子线程的runloop
CFRunLoopRef loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
    __CFUnlock(&loopsLock);
if (!loop) {
    // 如果子线程的runloop不存在,那么就为该线程创建一个对应的runloop
	  CFRunLoopRef newLoop = __CFRunLoopCreate(t);
        __CFLock(&loopsLock);
    loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
    // 把当前子线程和对应的runloop保存到字典中
    if (!loop) {
        CFDictionarySetValue(__CFRunLoops, pthreadPointer(t), newLoop);
        loop = newLoop;
    }
    // don't release run loops inside the loopsLock, because CFRunLoopDeallocate may end up taking it
        __CFUnlock(&loopsLock);
    CFRelease(newLoop);
    }
```

#### 2.2 Runloop 的销毁

RunLoop 的销毁发生在线程结束时。

### 3.Runloop 对外的接口

#### 3.1 Runloop 对象

* Foundation 框架    ——    `NSRunLoop 对象`

```objc
[NSRunLoop currentRunLoop]; // 获得当前线程的RunLoop对象
[NSRunLoop mainRunLoop]; // 获得主线程的RunLoop对象
```

* CoreFoundation 框架    ——    `CFRunLoopRef 对象`

```objc
CFRunLoopGetCurrent(); // 获得当前线程的RunLoop对象
CFRunLoopGetMain(); // 获得主线程的RunLoop对象
```

相关类：
	* CFRunLoopRef    ——    获取当前 Runloop 和 主 Runloop
	* CFRunLoopModeRef    ——    Runloop 运行模式。只能选择一种。不同模式下做不同的操作。
	* CFRunLoopSourceRef    ——    事件源、输入源
	* CFRunLoopTimerRef    ——    定时器时间
	* CFRunLoopObserverRef    ——    观察者

### 4.Runloop 逻辑

![](http://ouo0w16s7.bkt.clouddn.com/Runloop1.png?e=1515742302&token=X-k_MFo98a0VybdWmejDMbKtt0VLy2zl6vAXUB4r:EaWXeSQym0Y-bQPVjTY5cDOH9Ns)

**RunLoop 处理逻辑：**

* 1.通知观察者 run loop 已经启动
* 2.通知观察者将要开始处理 Timer 事件
* 3.通知观察者将要处理非基于端口的 Source0
* 4.启动准备好的 Souecr0
* 5.如果基于端口的源 Source1 准备好并处于等待状态，立即启动：并进入步骤 9
* 6.通知观察者线程进入休眠
* 7.将线程置于休眠直到任一下面的事件发生
	* （1）某一事件到达基于端口的源
	* （2）定时器启动
	* （3）Run loop 设置的时间已经超时
	* （4）run loop 被显式唤醒
* 8.通知观察者线程将被唤醒
* 9.处理未处理的事件，跳回 2
	* （1）如果用户定义的定时器启动, 处理定时器事件并重启 run loop。进入步骤 2
	* （2）如果输入源启动, 传递相应的消息
	* （3）如果 run loop 被显式唤醒而且时间还没超时, 重启 run loop。进入步骤 2
* 10.通知观察者 run loop 结束

### 5.Runloop 退出

**Runloop 退出条件：**
* 1.主线程销毁 RunLoop 退出
* 2.Mode 中有一些 Timer 、Source、 Observer，这些保证 Mode 不为空时保证 RunLoop 没有空转并且是在运行的，当 Mode 中为空的时候，RunLoop 会立刻退出
* 3.我们在启动 RunLoop 的时候可以设置什么时候停止
```objc
[NSRunLoop currentRunLoop]runUntilDate:<#(nonnull NSDate *)#>]
[NSRunLoop currentRunLoop]runMode:<#(nonnull NSString *)#> beforeDate:<#(nonnull NSDate *)#>]
```

### 6.Runloop 的使用

#### 6.1 常驻线程

给 Runloop 增加一个 NSTimer 或一个 Source 保证 Runloop 不会空转而退出，形成常驻线程。

#### 6.2 自动释放池

RunLoop 内部有一个自动释放池，当 RunLoop 开启时，就会自动创建一个自动释放池，当 RunLoop 在休息之前会释放掉自动释放池的东西，然后重新创建一个新的空的自动释放池。

主线程的 Runloop 会默认启动，也就意味着会自动创建一个自动释放池。
子线程的 Runloop 需要在线程调度方法中手动添加一个自动释放池。 

```objc
@autorelease{  
      // 执行代码 
}
```
