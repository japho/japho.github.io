---
layout:     post
title:      理解NSCopying协议
subtitle:   《Effective Objective-C 2.0》读书笔记
date:       2018-06-26
author:     Japho
header-img: postimage/post-bg-ioses.jpg
catalog:    true
tags:
    - iOS
    - 读书笔记
---

在需要拷贝对象时，要通过copy方法来完成，如果想令自己创建的类支持拷贝操作，那就需要实现NSCopying协议，次协议只有一个方法：

```
- (id)copyWithZone:(NSZone *)zone;
```

**关于zone**

开发时，会根据`zone`把内存分成不同的“区”，不过现在默认程序只有一个区：“默认区”，尽管必须实现此方法，不必担心`zone`参数。注意，这里的`copy`是由`NSObject`实现的，会以“默认区”为参数调用`copyWithZone:`，所以需要实现的是`copyWithZone:`方法，而非覆写`copy`。

**一般的：**

```
//特殊需要
@implentation Person
{
NSMutableSet *_friends;
}

- (id)copyWithZone:(NSZone *)zone
{
Person *copy = [[[self class] allocWithZone:zone] initWithFirstName:first andLastName:last];

/* 如果有特殊需要，例如保存特定的数据结构 */
copy->_friends = [_friends mutableCopy];

return copy;
}

```

使用“->”语法，因为这里`friends`变量并非属性，只是内部存在的一个成员变量。

这里拷贝了`friends`变量是防止原始数据修改后，导致新的复制过去的数据也会变化。若set对象是不可变的，这里则无需进行复制。

若要实现`mutableCopy`方法，同样的，遵守`NSMutableCopying`协议，实现`mutableCopyWithZone:`，
无论当前实例是否可变，若需要可变版本的拷贝，则调用`mutableCopy`，同理，需要不可变的拷贝，调用`copy`来获取。

**对于NSArry && NSMutableArray**

```
[NSMutableArray copy] => NSArray
[NSArray mutableCopy] => NSMutableArray
```

**编写拷贝时，需考虑一个问题：应该执行“深拷贝”还是“浅拷贝”？**

**深拷贝：**在拷贝对象自身时，将其底层数据也一并的复制过来。`Foundation`框架中所有的`collection`类在默认的情况下都执行浅拷贝，也就是说，只拷贝其容器本身，而不复制其中的数据。原因在于：容器内的对象未必都能拷贝，而调用者也未必想在拷贝容器时一并拷贝其中的每一个对象。

一般情况下，遵循系统框架的模式，对于自定义的类，以浅拷贝的方式实现`copyWithZone:`方法，必要的话，也可以增加一个深拷贝的方法。

例如`NSSet`类：

```
- (id)initWithSet:(NSArray *)array copyItems:(BOOL)copyItems;
```
若`copyItems`为`YES`执行深拷贝。

例如刚才的例子：

```
- (id)deepCopy
{
Person *copy = [[[self class] allocWithZone:zone] initWithFirstName:first andLastName:last];

/* 如果有特殊需要，例如保存特定的数据结构 */
copy->_friends = [[NSMutableSet alloc] initWithSet:_friends copyItems:YES];

return copy;
}
```

因为没有专门的深拷贝协议，所以具体执行方式有每个类来确定。只需决定自己缩写的类是有需要提供深拷贝方法就可以了。不要假定遵从了`NSCopying`协议的对象都会深拷贝。在绝大多数的情况下，都是执行浅拷贝的。如果需要在某个对象上执行深拷贝，那么除非另有说明，否则，要么找到能够找到执行深拷贝的相关方法，要么自己实现相关拷贝方法。

**要点**

- 若想令自己所写的类具有拷贝功能，则需实现`NSCopying`协议。
- 如果自定义的对象分为可变版本与不可变版本，那么就要同时实现`NSCopying` && `NSMutableCopying`协议。
- 复制对象时需决定采用深拷贝还是浅拷贝，一般情况下尽量执行浅拷贝。
- 如果所写的对象需要深拷贝，那么可以考虑新增一个专门执行深拷贝的方法。














