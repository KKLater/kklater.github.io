---
layout:         post
title:          iOS Runtime分析
subtitle:       
location:  北京 中国   
date:           2018-01-12 11:00:00
categories: iOS
tags:  iOS Runtime
excerpt:  最近面试，结合网络博客复习了下 Runtime 相关知识，做一下总结。本文简单介绍了什么是运行时，以及运行时在iOS开发过程中的几个常用场景。
---

## 1、什么是运行时？
要说运行时，还得从静态语言和动态语言说起。
静态语言的代码在编译期会经过编码器的分析，优化成机器语言，系统则按照设计的逻辑自上而下的执行代码。动态语言在编译期只是确定了要执行某行代码或程序，而只有在真正运行的时候才会实际执行代码或程序。
在OC里面采用消息机制实现动态执行，编译期只是知道要向某个对象发送消息，然而并没有实际发送，甚至于都不知道所需要执行的代码有没有具体实现。只有在运行期，才会向某个对象发送消息。
简单而言，所谓的“runtime”实际上就是一个管理运行代码的环境机制，保证了代码在运行中有自我检查，判断的能力。

## 2、“runtime”实际使用
### 2.1	发送消息
```objc
/**
	参数:
	1.消息接收对象
	2.消息对应的方法的标识SEL
	3.方法参数
*/
id objc_msgSend(id theReceiver, SELtheSelector, ...)
```

### 2.2	交换方法
**注意：**

#### 2.2.1 Swizzling应该总是在+load中执行
+load会在类初始加载时调用，+initialize会在第一次调用类的类方法或实例方法之前被调用。这两个方法是可选的，且只有在实现了它们时才会被调用。由于method swizzling会影响到类的全局状态，因此要尽量避免在并发处理中出现竞争的情况。**+load能保证在类的初始化过程中被加载，并保证这种改变应用级别的行为的一致性**。相比之下，+initialize在其执行时不提供这种保证—事实上，如果在应用中没为给这个类发送消息，则它可能永远不会被调用。

#### 2.2.2 Swizzling应该总是在dispatch_once中执行
因为swizzling会改变全局状态，故而需要**确保代码只被执行一次**。使用GCD的dispatch_once可以确保这种行为。
```objc
method_exchangeImplementations(newMethodNamed, oldMethodNamed)
```
```objc
#import <objc/runtime.h>
 
@implementation UIViewController (Tracking)
 
+ (void)load {
        static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];        
        // When swizzling a class method, use the following:
        // Class class = object_getClass((id)self);
 
        SEL originalSelector = @selector(viewWillAppear:);
        SEL swizzledSelector = @selector(xxx_viewWillAppear:);
 
        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
 
        BOOL didAddMethod =
                        class_addMethod(class,
                originalSelector,
                method_getImplementation(swizzledMethod),
                method_getTypeEncoding(swizzledMethod));
 
        if (didAddMethod) {
                        class_replaceMethod(class,
                swizzledSelector,
                method_getImplementation(originalMethod),
                method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}
 
#pragma mark - Method Swizzling
 
- (void)xxx_viewWillAppear:(BOOL)animated {
        [self xxx_viewWillAppear:animated];
    NSLog(@"viewWillAppear: %@", self);
}
 
@end
```
### 2.3	runtime 添加类方法
```objc
// 添加方法
class_addMethod(self, @selector(study), (IMP)study, "v@:")
```
```objc
void study(id reccevier, SEL sel) {
    
}

// 如果调用的方法没有实现，就会走这个方法
+ (BOOL)resolveInstanceMethod:(SEL)sel {
    if (sel == @selector(study)) {
        class_addMethod(self, @selector(study), (IMP)study, "v@:");
    }
    return [super resolveInstanceMethod:sel];
}
```

### 2.4	类目添加属性时，添加属性的setter、getter方法
```objc
// setter
objc_setAssociatedObject(self, key, score, OBJC_ASSOCIATION_ASSIGN)
// getter
objc_getAssociatedObject(self, key)
```
```objc
voidvoid *key;  
- (void)setScore:(NSNumber *)score {  
    // 关联引用  
    /** 
     *  1. 给哪个对象属性进行关联 
     *  2. 用来保存传入的值的指针 (用于 get 方法获取值) 
     *  3. 传入的值 (注意是对象类型) 
     *  4. 关联引用的策略 (这个根据属性添加的修饰而定) 
     */  
    objc_setAssociatedObject(self, key, score, OBJC_ASSOCIATION_ASSIGN);  
}  
- (NSNumber *)score {  
    id score = objc_getAssociatedObject(self, key);  
    return score;  
```

### 2.5	简洁高效实现归档、解档
解档归档可以使用kvc方式进行操作，但是当属性过多时，就显得繁琐且易出错。使用runtime进行归档、解档则简洁高效。

**操作步骤：**
```objc
// 获取类的所有成员变量
`Ivar *ivarList = class_copyIvarList([self class], &count)`
// 归档/解档
/// 获取成员变量的名称
const charchar *iVarName = ivar_getName(aIvar)
NSString *key = [NSString stringWithUTF8String:iVarName];
///	 获取成员变量值
id value = [self valueForKey:key];
```
```objc
// 归档  
- (void)encodeWithCoder:(NSCoder *)aCoder {
    // 获取某个类的所有成员变量
    unsigned int count = 0;
    Ivar *ivarList = class_copyIvarList([self class], &count);  
    // 归档
    for (int i = 0; i <count; i ++) {
        Ivar aIvar = ivarList[i];
        // 获取成员变量的名称
        const charchar *iVarName = ivar_getName(aIvar);  
        id value = [self valueForKey:[NSString stringWithUTF8String:iVarName]];
        if (!value) {
            
        } else {
            [aCoder encodeObject:value forKey:[NSString stringWithUTF8String:iVarName]];
        }
    }
}

// 解档  
- (instancetype)initWithCoder:(NSCoder *)aDecoder {
    if (self = [super init]) {
        unsigned int count = 0;
        Ivar *ivarList = class_copyIvarList([self class], &count);
        for (int i = 0; i <count; i ++) {
            Ivar aIvar = ivarList[i];
            const charchar *name = ivar_getName(aIvar);
            id value = [aDecoder decodeObjectForKey:[NSString stringWithUTF8String:name]];
            if (!value) {
                  
            }else {
                [self setValue:value forKey:[NSString stringWithUTF8String:name]];
            }
        } 
    }  
    return self;  
}  
```

### 2.6	runtime实现字典模型转换
```objc
// 获取属性列表
objc_property_t *propertyList = class_copyPropertyList([self class], &count)
// 获取属性名
objc_property_t aProperty = propertyList[i];
const char *name = property_getName(aProperty)
// 使用kvc给属性赋值
[aStudent setValue:value forKey:[NSString stringWithUTF8String:name]]
```
```objc
+ (instancetype)modelWithDictionary:(NSDictionary *)dic {
    Student *aStudent = [Student new];
    unsigned int count = 0;
	  // 获取属性列表
    objc_property_t *propertyList = class_copyPropertyList([self class], &count);
    for (int i = 0; i < count; i ++) {
        objc_property_t aProperty = propertyList[i];
        //获取属性名
        const char *name = property_getName(aProperty);
        
        id value = dic[[NSString stringWithUTF8String:name]];
        if (!value) {
            
        }else {
            //使用kvc给属性赋值
            [aStudent setValue:value forKey:[NSString stringWithUTF8String:name]];
        }
    }
    return aStudent;
}
```
