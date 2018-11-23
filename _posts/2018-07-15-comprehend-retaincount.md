---
layout:     post
title:      理解引用计数
subtitle:   《Effective Objective-C 2.0》读书笔记
date:       2018-07-15
author:     Japho
header-img: https://ws2.sinaimg.cn/large/006tNbRwly1fxi1gfa392j31gs0o6jv6.jpg
catalog:    true
tags:
    - iOS
    - 读书笔记
---

Objective-C语言使用引用技术来管理内存，也就是说，每个对象都有可以递增或递减的计数器。如果想使某个对象继续存活，就递增其引用计数；用完之后，就递减其引用计数，计数变为0时，表示没有人关注这个对象，自然就把他释放了。

### 引用计数工作原理

引用计数的架构下，对象有个计数器，用以表示当前有多少事物想令此对象存活下去，在Objective-C中叫做“引用计数”。`NSObject`协议中声明了一下三个方法用于操作计数器，以递增或递减引用计数。

- `retain` 递增引用计数；
- `release` 递减引用计数；
- `autorelease` 待稍后清理“自动释放池”时，再去递减其引用计数。

查看引用计数的方法叫做`retainCount`。

对象在创建出来时，其引用计数至少为1，若想继续存活，则调用`retain`方法。要是某部分代码不再调用此对象，不想令其存活，则调用`release`或`autorelease`方法。当引用计数为0时，对象就会被回收。这时，系统会将其占用的内存标记为“可重用”（`reuse`）。此时所有指向该对象的引用都变得无效了。

下列代码：

```
NSMutableArray *array = [[NSMutableArray alloc] init];
NSNumber *number = [[NSNumber alloc] initWithNumber:1337];
[array addObject:number];
[number release];
//do somthing with array;
[array release];
```

由于上述代码调用了`release`，所以在ARC下无法编译。在`Objective-C`中，调用`alloc`方法所返回的对象由调用者所有。也就是说，调用者，已通过alloc方法表达了相对该对象继续存活的意愿。这里可以肯定此时引用计数一定大于1。
创建完成后，把`number`加入数组中，调用`addObject:`方法时，数组也会在`number`上调用`retain`方法，以继续保留此对象。此时，引用计数至少为2。接下来，代码不再需要`number`对象了，于是将其释放，此时引用计数至少为1，这样就不能照常使用`number`变量了。调用`release`之后，已经无法保证所指对象是否仍然存活。不过这里我们知道`array`还对`number`存在引用。但是不要假设这一对象一定存活，也就是说，不要编写下列代码：

```
NSMutableArray *array = [[NSMutableArray alloc] init];
NSNumber *number = [[NSNumber alloc] initWithNumber:1337];
[array addObject:number];
[number release];
NSLog(@"number = %@",number);
```

上述代码仍可执行，但如果调用`release`之后，基于某种原因，引用计数变为0，那么`number`对象所占有内存也会被回收，这样，再去调用`NSLog`就可能造成崩溃。这里只是说“可能”，并不是“一定”，因为对象所占有的内存在“解除分配”之后，只是放回“可用内存池”（`avaiable pool`）。如果执行`NSLog`时尚未覆写对象内存，那么该对象仍然有效，这是程序并不会崩溃，不过过早释放对象而造成的bug将会很难调试。

为了避免在不经意间使用了无效对象，一般调用完`release`之后都会进行清空指针。这就能保证不会出现可能指向无效指针对象的指针。（悬挂指针）。例如：

```
NSNumber *number = [[NSNumber alloc] initWithNumber:1337];
[array addObject:number];
[number release];
number = nil;
```

### 属性存取方法中的内存管理

访问属性时，会用到相关实例变量的`getter`方法及`setter`方法。若属性为`strong`关系，则设置的属性值会保留，例如：

```
- (void)setFoo:(id)foo
{
[foo retain];
[_foo release];
_foo = foo;
}
```

此方法将保留新值并释放旧值，然后更新实例变量。

### 自动释放池

调用`releas`e会立刻递减对象的保留计数，有时可能令系统回收此对象，然后有时候改为调用`autorelease`，此方法会在稍后递减计数，通常是在下一次“事件循环”时递减，不过也可能会更早。
`autoreleas`e能延长对象生命周期，使其在跨越方法调用便捷后依然可以存活一段时间。

### 循环引用

呈环状互相引用多个对象，这将导致内存泄漏。因为循环中的对象其引用计数不会降为0，对于循环中的每个对象来说，至少还有另外一个对象引用他。通常采用“弱引用”解决，或者从外界命令循环中的某个对象不再保留另外一个对象。

**要点：**

- 引用计数机制通过可以递增递减的计数器来管理内存。对象创建好之后，其引用计数至少为1。若引用计数为正，则对象继续存活，当引用计数降为0时，对象就被销毁了。
- 在对象生命周期中，其余对象通过引用来保留或者释放此对象。保留与释放操作分别会递增及递减引用计数。
