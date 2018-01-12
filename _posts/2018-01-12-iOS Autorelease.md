---
layout:         post
title:          iOS AutoreleasePool 分析
subtitle:       
location:  北京 中国   
date:           2018-01-12 11:30:00
categories: iOS
tags:  iOS Autorelease
excerpt:  autorelease 是把对象放入 autoreleasePool 中，待autoreleasePool 释放时，该自动释放池中的对象才会被……
---

# AutoReleasePool

* 参考：
  [黑幕背后的Autorelease · sunnyxx的技术博客](http://blog.sunnyxx.com/2014/10/15/behind-autorelease/)

## 1.Autorelease 与 Release 区别
​	Release 只是把对象引用计数 - 1。autorelease 是把对象放入 autoreleasePool 中，待autoreleasePool 释放时，该自动释放池中的对象才会被调用 release 。


## 2.AutoreleasePool 原理



`AutoReleasePool` 使用 `AutoReleasePoolPage` 这个类来实现自动释放机制，`AutoReleasePoolPage` 也是自动释放机制的核心。

> * AutoreleasePool 并没有单独的结构，而是由若干个 AutoreleasePoolPage 以双向链表的形式组合而成（分别对应结构中的 parent 指针和 child 指针）
> * AutoreleasePool 是按线程一一对应的
> * AutoreleasePoolPage 每个对象会开辟 4096 字节内存（也就是虚拟内存一页的大小），用于存储 AutoreleasePoolPage 实例变量和储存 autorelease 对象的地址
> * AutoreleasePoolPage 实例对象中，游标指针指向栈顶最新 add 进来的 autorelease 对象的下一个位置
> * 一个 AutoreleasePoolPage 的空间被占满时，会新建一个 AutoreleasePoolPage 对象，连接链表，后来的 autorelease 对象在新的 page 加入。

### 2.1 AutoReleasePool 添加与清理

@autoreleasepool{} 的实质是：
```objc
// push 时插入哨兵对象，存储哨兵对象地址
void *context = objc_autoreleasePoolPush();
// {}中的代码
// pop 时获取哨兵对象地址
objc_autoreleasePoolPop(context);
```

> * 1.每当调用一次 @autoreleasepool{} 来使用 AutoreleasePool ，运行时会向当前的 AutoreleasePoolPage 中添加一个值为 0 的哨兵对象。
> * 2.向一个对象发送 `- autorelease` 消息，就是将这个对象加入到当前 AutoreleasePoolPage 的栈顶 next 指针指向的位置。
> * 3.当当前 runloop 迭代时，autoreleasePool 释放，则根据哨兵对象地址找到哨兵对象，并向晚于哨兵对象插入的所有 autorelease 对象发送一次 `- release` 消息，如果引用计数为0，则该对象释放。回移 `next` 指针指向正确的地址。
> * 4.从最新加入的 autorelease 对象一直向前清理，可以向前跨越多个page，直到哨兵位置。

## 3.AutoreleasePool 与 Runloop

​	在App 启动后，苹果在主线程 #RunLoop 里注册了两个 Observer。

​	第一个 Observer 监视的事件是 Entry(即将进入 Loop)，其回调内会调用 `_objc_autoreleasePoolPush()` 创建自动释放池。其 order 是 - 2147483647，优先级最高，保证创建释放池发生在其他所有回调之前。

​	第二个 Observer 监视了两个事件： BeforeWaiting(准备进入休眠) 时调用`_objc_autoreleasePoolPop()` 和 ` _objc_autoreleasePoolPush()` 释放旧的池并创建新池；Exit(即将退出 Loop) 时调用 `_objc_autoreleasePoolPop()` 来释放自动释放池。这个 Observer 的 order 是 2147483647，优先级最低，保证其释放池子发生在其他所有回调之后。


## 4.什么时候使用自动释放池
* 循环中创建了许多临时对象，在循环里使用自动释放池，用来减少高内存占用
* 开启子线程的时候要自己创建自己的释放池，否则可能会发生内存泄漏

## 5.注意
使用容器的 block 版本的枚举器时，内部会自动添加一个 AutoreleasePool。在普通 for 循环和 for in 循环中没有，所以，还是新版的 block 版本枚举器更加方便。for 循环中遍历产生大量 autorelease 变量时，就需要手加局部 AutoreleasePool。
