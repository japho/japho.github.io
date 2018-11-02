---
layout:     post
title:      UIButton传递多个参数的方法
subtitle:   使用关联函数来解决这个问题
date:       2016-07-16
author:     Japho
header-img: postimage/post-bg-swift2.jpg
catalog:    true
tags:
    - iOS
---

## 前言

在平时开发时经常会要遇到通过button的绑定值来做逻辑处理以区分不同的button，通常使用tag来区分，但是当需要传多个值的时候这就比较麻烦了，通常考虑用全局变量来传值，今天来介绍另一种给`UIButton`传值的方法——关联函数。下面来简单介绍下关联。

## 关联

关联是指把两个对象相互关联起来，使得其中的一个对象作为另外一个对象的一部分。
关联特性只有在Mac OS X V10.6以及以后的版本上才是可用的。

## 在类的定义之外为类增加额外的存储空间

使用关联，我们可以不用修改类的定义而为其对象增加存储空间。这在我们无法访问到类的源码的时候或者是考虑到二进制兼容性的时候是非常有用。

关联是基于关键字的，因此，我们可以为任何对象增加任意多的关联，每个都使用不同的关键字即可。关联是可以保证被关联的对象在关联对象的整个生命周期都是可用的（在垃圾自动回收环境下也不会导致资源不可回收）。

## 创建关联

创建关联要使用到Objective-C的运行时函数：`objc_setAssociatedObject`来把一个对象与另外一个对象进行关联。该函数需要四个参数：源对象，关键字，关联的对象和一个关联策略。当然，此处的关键字和关联策略是需要进一步讨论的。

- 关键字是一个void类型的指针。每一个关联的关键字必须是唯一的。通常都是会采用静态变量来作为关键字。
- 关联策略表明了相关的对象是通过赋值，保留引用还是复制的方式进行关联的；还有这种关联是原子的还是非原子的。这里的关联策略和声明属性时的很类似。这种关联策略是通过使用预先定义好的常量来表示的。

下面的代码展示了如何把一个字符串关联到一个数组上。
注意使用时导入头文件` <objc/runtime.h>`
`objc_setAssociatedObject`需要四个参数：源对象，关键字，关联的对象和一个关联策略。

>1、源对象alert
>
>2、关键字 唯一静态变量key associatedkey
>
>3、关联的对象 sender
>
>4、关键策略  `OBJC_ASSOCIATION_RETAIN_NONATOMIC`

```swift
static char overviewKey;    
NSArray * array =[[NSArray alloc] initWidthObjects:@"One", @"Two", @"Three", nil nil];    
NSString * overview = @"First three numbers";    
objc_setAssociatedObject(array, &overviewKey, overview, OBJC_ASSOCIATION_RETAIN_NONATOMIC); 
```

## 获取关联对象

获取相关联的对象时使用Objective-C函数`objc_getAssociatedObject`。接着上面的代码，我们可以使用如下代码来获取与array相关联的字符串：

```swift
NSString * associatedObject = (NSString *)objc_getAssociatedObject(array, &oveviewKey);
```

## 断开关联

断开关联是使用`objc_setAssociatedObject`函数，传入`nil`值即可。接着上面的程序，我们可以使用如下的代码来断开字符串`overview`和`arry`之间的关联

```swift
objc_setAssociatedObject(array, &overviewKey, nil, OBJC_ASSOCIATION_ASSIGN);
```

其中，被关联的对象为`nil`，此时关联策略也就无关紧要了。使用函数`objc_removeAssociatedObjects`可以断开所有关联。通常情况下不建议使用这个函数，因为他会断开所有关联。只有在需要把对象恢复到“原始状态”的时候才会使用这个函数。

## 实现UIButton传值

通过btn传递两个实例对象`firstObject`和`secondObject`，创建button并关联对象及button的点击事件

```swift
UIButton *btn = // create the button  
objc_setAssociatedObject(btn, "firstObject", someObject, OBJC_ASSOCIATION_RETAIN_NONATOMIC);  
objc_setAssociatedObject(btn, "secondObject", otherObject, OBJC_ASSOCIATION_RETAIN_NONATOMIC);  
[btn addTarget:self action:@selector(click:) forControlEvents:UIControlEventTouchUpInside];  
  
- (void)click:(UIButton *)sender  
{  
    id first = objc_getAssociatedObject(btn, "firstObject");  
    id second = objc_setAssociatedObject(btn, "secondObject");  
    // etc.  
}
```

向以上方法那样，只要设置了关联，当把button传递给@selector的参数时，也顺便把另外两个参数也传递过去了。就类似设了button的两个属性，但是这种方法简单多了。

- 第一步: 设定关联`objc_setAssociatedObject(btn, "firstObject", someObject, OBJC_ASSOCIATION_RETAIN_NONATOMIC);`第一个参数：btn为被关联者(主体)，第二个参数：`firstObject`为指向关联的对象的指针(一般为一个`static`字符串)，第三个参数：关联的对象实例，第四个对象：关联的方式(有几种，类似设定.h文件属性时候的 `assign`,`retain`等)
- 第二步：得到关联的对象
`id first = objc_getAssociatedObject(btn, "firstObject");`第一个参数：为被关联者第二个参数：为关联对象的指针。

参考：[http://stackoverflow.com](http://stackoverflow.com)