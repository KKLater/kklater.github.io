---
layout:         post
title:          iOS 泛型（Lightweight Generice）的使用
subtitle:       Lightweight Generice
location:  北京 中国   
date:           2016-04-27 21:30:00
categories: iOS
tags:  Objective-c
excerpt:  Lightweight Generics 轻量级泛型，纯编译器的语法支持，不借助任何objc runtime的升级。即，完全向下兼容（兼容更低的iOS版本）。
---
`Lightweight Generics` 轻量级泛型，纯编译器的语法支持，不借助任何objc runtime的升级。即，完全向下兼容（兼容更低的iOS版本）。

###带泛型的容器

之前使用容器时，常用写法：

```objc
 NSArray * strings = @[@"Hello",@"word"];
```

在使用时，例如调用

```objc
[strings enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
     //obj,  不涉及类型，需要自行转换类型
     NSString *first = (NSString *)obj;
    NSLog(@"first = %@",first);//Hello
}];
```

###引入泛型之后的容器

```objc
 NSArray<NSString *> * strings = @[@"Hello",@"word"];
```

在使用时，调用：

```objc
[strings enumerateObjectsUsingBlock:^(NSString * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
    //obj, 自动设置为NSString类型，不需要转换类型
    NSLog(@"first = %@",obj);
}];
```

`NSMutableArray`、`NSDictionary`、`NSMutableDictionary`类似。

###自定义泛型类

除了系统的泛型容器，我们还可以自定义泛型类。例如我们自定义一个`FBTObject`类。

```objc
    #import <Foundation/Foundation.h>
    @interface FBTObject<ObjectType> : NSObject
    @property (nonatomic, readonly) NSMutableArray <ObjectType> *allObjects;
    - (void)saveObject:(ObjectType)object;
    @end
```

这里的`<ObjectType>`引入一个泛型。它只能在`@interface`上定义（类声明、类扩展、Category等），特可以是任意定义的非关键字字符。这个类型只在`@interface`和`@end`区间的作用域有效。
使用：
若是创建对象时不指定泛型类型，例如：

```objc
    FBTObject *fbt = [[FBTObject alloc] init];
```

则在使用该对象时，不会引入泛型。

![](http://ouo0w16s7.bkt.clouddn.com/%E4%B8%8D%E6%8C%87%E5%AE%9A%E6%B3%9B%E5%9E%8B%E7%9A%84%E8%87%AA%E5%AE%9A%E4%B9%89%E5%AF%B9%E8%B1%A1.png?e=1505125324&token=tYezop22TZHJ2SRLGu2c2pc_DIsOrZZ3JfZjeV5-:YcO7l7mn2_9Mj720_e-9oDcuXrU)

在创建了泛型的对象，例如：

```objc
    FBTObject <NSString *> *fbt = [[FBTObject alloc] init];
```

则在使用该对象时，会自动引入指定泛型类型：
![](http://ouo0w16s7.bkt.clouddn.com/%E6%8C%87%E5%AE%9A%E4%BA%86%E6%B3%9B%E5%9E%8B%E7%B1%BB%E5%9E%8B%E7%9A%84%E5%AF%B9%E8%B1%A1%E7%9A%84%E4%BD%BF%E7%94%A8.png?e=1505125325&token=tYezop22TZHJ2SRLGu2c2pc_DIsOrZZ3JfZjeV5-:8SHNR0S9nAVdEtNNZ0VuJnQvEow)

同样，我们可以直接在对象声明中指定泛型的类型。

```objc
    @interface FBTObject<ObjectType: NSNumber *> : NSObject
```

使用时不需要再指定泛型类型，即可自动识别泛型类型：
![](http://ouo0w16s7.bkt.clouddn.com/%E5%A3%B0%E6%98%8E%E6%97%B6%E6%8C%87%E5%AE%9A%E4%BA%86%E9%BB%98%E8%AE%A4%E6%B3%9B%E5%9E%8B%E7%B1%BB%E5%9E%8B%E7%9A%84%E4%BD%BF%E7%94%A8.png?e=1505125324&token=tYezop22TZHJ2SRLGu2c2pc_DIsOrZZ3JfZjeV5-:VvO_3S5W9NxZU65hfSxUDfW05f8)
在声明中指定了泛型类型，我们还可以重新指定范型类型吗？我们试试看！
我们上面的例子已经指定了`FBTObject`的默认泛型类型为`NSNumber`类，我们重新指定泛型为`<NSString *>`，发现出现下面的错误提醒：
![](http://ouo0w16s7.bkt.clouddn.com/%E9%87%8D%E6%96%B0%E6%8C%87%E5%AE%9A%E6%B3%9B%E5%9E%8B%E7%9A%84%E9%94%99%E8%AF%AF.png?e=1505125326&token=tYezop22TZHJ2SRLGu2c2pc_DIsOrZZ3JfZjeV5-:L7N_6Od5fnzbrir71b8qGX67aTY)
那么，真的不能重新指定泛型了么？看下面的例子。
我们在`FBTObject`声明中重新指定泛型类型为NSString类，在创建对象时重新指定泛型类型为`NSMutableString`类。

```objc
    #import <Foundation/Foundation.h>
    @interface FBTObject<ObjectType: NSString *> : NSObject
    @property (nonatomic, readonly) NSMutableArray <ObjectType> *allObjects;
    - (void)saveObject:(ObjectType)object;
    @end
```

![](http://ouo0w16s7.bkt.clouddn.com/%E9%94%99%E8%AF%AF%E7%9A%84%E9%87%8D%E6%96%B0%E6%8C%87%E5%AE%9A%E6%B3%9B%E5%9E%8B.png?e=1505125326&token=tYezop22TZHJ2SRLGu2c2pc_DIsOrZZ3JfZjeV5-:GeWvfgfe-A9d4EIAlY1Oh83aIaM)
发现没有出现上面的错误提醒，为什么呢？
我们再重新实验下，声明中指定泛型类型为`NSMutableString`，创建对象时指定泛型类型为`NSString`类，看`Xcode`有没有给我们错误提醒。
我们发现，红色的错误提醒又出现了...
那么为什么指定了`NSString`泛型的，重新指定`NSMutableString`可以，反过来就错了？
我们发现，其实`NSMutableString`是`NSString`的子类！

###协变性和逆变性

我们再来讨论一个有意思的事情。`FBTObject`不指定泛型类型的情况下，我定义了三个`FBTObject`对象：

```objc
        FBTObject *fbt = [[FBTObject alloc] init];
        FBTObject <NSString *> *fbtString = [[FBTObject alloc] init];
        FBTObject <NSMutableString *> *fbtMutableString = [[FBTObject alloc] init];
```

那么，`fbt`和`fbtString`、`fbtMutableString`能直接赋值么？像下面这样：

```objc
        fbt = fbtString;
        fbt = fbtMutableString;
        
        fbtString = fbtMutableString;
        fbtMutableString = fbtString;
```

我们发现，前面的赋值是没有问题的，而后面的两个赋值都出现了黄色的警告提醒。
![](http://ouo0w16s7.bkt.clouddn.com/%E6%B3%9B%E5%9E%8B%E5%AF%B9%E8%B1%A1%E8%B5%8B%E5%80%BC%E8%AD%A6%E5%91%8A.png?e=1505125326&token=tYezop22TZHJ2SRLGu2c2pc_DIsOrZZ3JfZjeV5-:zRyaYguUB_xlIL57WIx-Jk85H4c)
那么这是为什么呢？
我们知道`fbt`是没有指定泛型类型的对象，相当于泛型类型为id,可以赋值指定任意泛型类型的对象，就像

```objc
        id fbt = [[NSObject alloc] init];
        NSString *fbtString = [[NSString alloc] init];
        NSMutableString *fbtMutableString = [[NSMutableString alloc] init];
```

`Xcode`编译器允许我们这样赋值

```objc
        fbt = fbtString;
        fbt = fbtMutableString;
```

而`fbtString`和`fbtMutableString`是制定了泛型的，直接赋值时Xcode编译器需要知道哪些赋值是允许的，哪些赋值是不允许的，也就出现了我们所说的协变性和逆变性。

`__covariant`- 协变性，子类型可以强转到父类型（里氏替换原则）

`__contravariant` - 逆变性，父类型可以强转到子类型（`WTF`？）

我们来试试！

在声明时加上`__covariant`看什么效果：

```objc
    #import <Foundation/Foundation.h>
    @interface FBTObject<__covariant ObjectType> : NSObject
    @property (nonatomic, readonly) NSMutableArray <ObjectType> *allObjects;
    - (void)saveObject:(ObjectType)object;
    @end
```

![](http://ouo0w16s7.bkt.clouddn.com/%E5%8D%8F%E5%8F%98%E6%80%A7.png?e=1505125324&token=tYezop22TZHJ2SRLGu2c2pc_DIsOrZZ3JfZjeV5-:fAmMkUHxKev9CYzGPMeXwkxu3kg)
子类型可以强制赋值给父类型了，我们竟然没有写强制转换！
在声明时加上`__contravariant`试试呢？

```objc
    #import <Foundation/Foundation.h>
    @interface FBTObject<__contravariant ObjectType> : NSObject
    @property (nonatomic, readonly) NSMutableArray <ObjectType> *allObjects;
    - (void)saveObject:(ObjectType)object;
    @end
```

![](http://ouo0w16s7.bkt.clouddn.com/%E9%80%86%E5%8F%98%E6%80%A7.png?e=1505125326&token=tYezop22TZHJ2SRLGu2c2pc_DIsOrZZ3JfZjeV5-:HLiH6A-QKUXb9hHgUk4O30cRHRY)
我们发现，父类型强制赋值给了子类型，我们也没有写强制转换！
那么，有没有方式即能实现父类强制赋值给子类型，又能实现子类型强制赋值给父类型呢？我们把`__covariant`和`__contravariant`都加上试试看。
又出现了错误警告，课件Xcode编译器是不允许他们同时存在的！其实一般情况下协变性是常见的，像`NSArray`的泛型，而逆变性我们基本不会用到。



