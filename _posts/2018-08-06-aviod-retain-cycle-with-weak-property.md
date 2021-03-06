---
layout:     post
title:      以弱引用避免保留环
subtitle:   《Effective Objective-C 2.0》读书笔记
date:       2018-08-06
author:     Japho
header-img: postimage/post-bg-desk.jpg
catalog:    true
tags:
    - iOS
    - 读书笔记
---

对象中经常出现一种情况，就是几个对象都以某种方式互相引用，从而形成了“环”。这种情况通常会造成内存泄漏，因为最后没有别的东西会引用环中的对象。这样的话，环里的对象就无法为外界所访问了，但对象间尚有引用，这使得他们都能够存活下去，而不会被系统回收。

循环引用会导致内存泄漏，从一开始编码时，就应该注意别出现循环引用。

避免循环引用的最佳方式就是使用弱引用。这种引用通常来表示“非拥有关系”。将属性声明为`unsafe_unretained`即可。

属性中的`unsafe_unretained`表示，属性值可能不安全，而且不归此实例拥有。如果系统已经属性所指的那个对象回收了，那么再其上调用方法可能会使应用崩溃。用`unsafe_unretained`修饰的属性，其语义同`assign`等价，然后assign通常只修饰“基本类型”，`unsafe_unretained`多用于对象类型。
ARC中还有一种弱引用特质，就是weak，他与`unsafe_unretained`的作用完全相同。然而，系统把属性回收，属性值就会自动设置为nil。
如果对象已经被回收，引用他的实例仍然存活，这就是编程错误。使用weak而非`unsafe_unretained`引用可令代码更加安全。

**要点：**

- 将某些引用设为weak，可避免出现“循环引用”
- weak引用可以自动清空，也可不自动清空。自动清空是随着ARC而引入的新特性，由运行期系统实现。在具备自动清空的弱引用上，可以随意读取其数据，因为这种引用不会指向已经回收过的对象。
