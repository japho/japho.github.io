---
layout:     post
title:      使用Block需注意的问题 
subtitle:   Block,了解一下？
date:       2016-08-18
author:     Japho
header-img: postimage/post-bg-swift.jpg
catalog:    true
tags:
    - iOS
---

**1、block中引用外部变量**

block中引用外部变量时，通常会把对象当做常量变量编码到block中，并且在block中尝试改变外部变量的值会报错，解决办法是引入`__block`标识符，需要在block内部修改的变量标识为`__block scope`

**2、block自身的内存管理**

block本身是像对象一样可以retain，和release。但是，block在创建的时候，它的内存是分配在栈(stack)上，而不是在堆(heap)上。他本身的作于域是属于创建时候的作用域，一旦在创建时候的作用域外面调用block将导致程序崩溃。解决的方法就是在创建完block的时候需要调用copy的方法。copy会把block从栈上移动到堆上，那么就可以在其他地方使用这个block了:`_block = [_block copy];`

**3、循环引用**

在block创建中：

```
_block = ^( ) { NSLog:(@“string %@“,_string ) }; 
```

`_string`是成员变量
这里的`_string`相当于是`self->_string`；那么block是会对内部的对象进行一次retain。也就是说，self会被retain一次。当self释放的时候，需要block释放后才会对self进行释放，但是block的释放又需要等self的dealloc中才会释放。如此一来变形成了循环引用，导致内存泄露。修改方案是新建一个`__block scope`的局部变量，并把self赋值给它，而在block内部则使用这个局部变量来进行取值。因为`__block`标记的变量是不会被自动retain的。

```
__block ViewController *controller = self;
_block = ^(){NSLog(@"string %@", controller->_string); };
```
