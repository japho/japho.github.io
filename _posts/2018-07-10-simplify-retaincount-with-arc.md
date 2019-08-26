---
layout:     post
title:      以ARC简化引用计数
subtitle:   《Effective Objective-C 2.0》读书笔记
date:       2018-07-10
author:     Japho
header-img: postimage/post-bg-alibaba.jpg
catalog:    true
tags:
    - iOS
    - 读书笔记
---

使用ARC时，引用计数实际上还是要执行的，只不过保留与释放操作现在是由ARC自动添加的。由于ARC会自动执行`retain`，`release`，`autorelease`等操作，所以直接在ARC下调用这些内存管理方法是非法的。

**例如以下方法：**

- retain
- release
- autorelease
- dealloc
 
直接调用这些方法会产生编译错误。

### 使用ARC时必须遵循的方法命名规则

若以以下词语开头，其返回的对象归调用者所有：

- alloc
- new
- copy
- mutableCopy

**归调用者所有的意思是：**

>调用上述四种方法的那段代码要负责释放方法所返回的对象。也就是说，这些对象的保留计数是正值，而调用了这四种方法的那段代码要将其中一次保留操作抵消掉。
>
>除了自动调用“保留”与“释放”的方法之外，使用ARC时，他可以在编译期把能够相互抵消的`retain`、`release`、`auitorelease`操作约简。如果发现在同一个对象上执行了多次“保留”与“释放”操作，那么ARC有时也可以成对的移除这两个操作。

### 变量的内存管理语义

ARC会用一种安全的方式来设置：先保留新值，再释放旧值，最后设置实例变量。使用ARC之后，无需考虑这种“边界情况”。

在应用程序中，可用下列修饰符来改变局部变量与实例变量的语义：

- `__strong`：默认语义，保留这个值
- `__unsafe_unretained`：不保留这个值，（可能不安全，下次使用变量时，其对象可能已经被回收）
- `__weak`：不保留这个值，但是变量可以安全使用，因为如果系统把这个对象回收了，那么变量也会被自动清空。
- `__autoreleasing`：把对象“按引用传递”给方法时，使用这个特殊的修饰符，这个值在方法返回时自动释放。

例如：想令实力变脸的语义与不使用ARC时相同，可以运用`__weak`或`__unsafe_unretained`修饰符。

```
@interface EOCClass  :  NSObject
{
id __weak _weakObject;
id __unsafe_unretained  _unsafeUnretainedObject;
}
```

### ARC如何清理实例变量

非Objective-C的对象，比如`CoreFoundation`框架中的对象，或是由`malloc()`分配在堆中的内存，这时仍然需要手动清理。这里调用`dealloc`方法时，并不需要调用超类的`dealloc`，在ARC下不能直接调用`dealloc`。ARC会自动生成代码并运行此方法，生成的代码中会自动调用超类的`dealloc()`方法。可以这么写：

```
-  (void)dealloc
{
CFRelease(_coreFoundationObject);
free(_heapAllocatedMemoryBlob);
}
```

### 覆写内存管理方法

不使用ARC时，可以覆写内存管理方法，但是在ARC环境下，不能这么做，因为会干扰ARC分析对象生命周期的工作。

**要点：**

- 有ARC之后，程序员就无须担心内存管理问题了。使用ARC来编程，可省去勒种的许多“样板代码”。
- ARC管理对象生命周期的办法基本上就是：在合适的地方插入“保留”及“释放”操作。在ARC环境下，变量的内存管理语义可以通过修饰符指明，而原来则需要手动执行“保留”及“释放”操作。
- 由方法所返回的对象，其内存管理语义总是通过方法名来体现。ARC将此确定为开发者必须遵守的规定。
ARC只负责管理Objective-C对象的内存。`CoreFoundation`对象不归ARC管理，开发者需要在合适的时间调用`CFRetain`/`CFRelease`。
