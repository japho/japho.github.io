---
layout:     post
title:      在dealloc方法中只释放引用并解除监听
subtitle:   《Effective Objective-C 2.0》读书笔记
date:       2018-07-02
author:     Japho
header-img: postimage/post-bg-android.jpg
catalog:    true
tags:
    - iOS
    - 读书笔记
---

对象经历生命周期后，就会被系统回收，这时就会执行`dealloc`方法。在每个对象的生命周期内，此方法只执行一次。不应该自己调用`dealloc`方法，运行期系统会在适当的时候调用他，而且一旦调用`dealloc`之后，对象就不在有效了，后续调用的方法均是无效的。

在`dealloc`方法中主要是释放对象所拥有的引用，把所有的Objective-C对象都释放掉，ARC会在`dealloc`中自动添加这些释放代码。对象所拥有的其他非Objective-C对象也要释放，例如`CoreFoundation`对象就必须手动进行释放，因为他是基于C的API生成的。

在`dealloc`方法中，通常还需要将原来配置过的监听方法都清理掉。如果用`NSNotificationCenter`给对象注册了某种通知，那么一般应该在这里注销，这样，通知系统就不会再把通知发送给回收后的对象了，如果还发送通知，则必然会引起程序崩溃。

**例如：**

```
- (void)dealloc
{
CFRelease(coreFoundationObject);
[[NSNotificationCenter  defaultCenter]  removeObserver:self];
/*
**    如果非ARC情况下，还需调用
[super dealloc];
*/
}
```

调用`dealloc`方法的那个线程会执行“最终的释放操作”，令对象的保留计数降为0，而某些方法必须在特定的线程中调用（主线程）。若在`dealloc`调用了哪些方法，则无法保证当前这个线程就是那些方法所需的线程。通过正常的代码编写方式，无论如何都无法保证其会安全运行在正确的线程上，因为对象正处于“正在回收的状态”，为了指明此状况，运行期系统已经改动了对象的内部数据结构。

在`dealloc`里也不要调用属性的`setter`、`getter`方法，因为这些方法可能会被重写，并与其中做了一些无法在回收阶段安全执行的操作。此外，属性可能处于“键值观察”机制的监控下，该属性的观察者可能会在属性值改变时“保留”或者使用这个即将回收的对象，这会令程序状态失调，导致一些莫名其妙的错误。

**要点：**

- 在`dealloc`方法里，应该做的事情就是释放指向其他对象的引用，并取消原来注册的`KVO`或`NSNotificationCenter`等通知，不要做其他事情。
- 如果对象持有文件描述符等系统资源，那么应该专门编写一个方法来释放这种资源。这样的类要和其使用者的约定：在用完资源之后必须调用`close`方法。
- 执行异步任务的方法不应该在`dealloc`里调用，只能在正常状态下执行的那些方法也不应该在`dealloc`里调用，因为此时对象已经处于正在回收的状态了。