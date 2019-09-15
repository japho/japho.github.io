---
layout:     post
title:      勿在分类中声明属性
subtitle:   《Effective Objective-C 2.0》读书笔记
date:       2018-07-01
author:     Japho
header-img: postimage/post-bg-coffee.jpg
catalog:    true
tags:
    - iOS
    - 读书笔记
---

尽管在技术上说，分类中可以声明属性，但是还是要尽量避免这种做法。因为，分类无法向类中新增实例变量，因此，他们无法把实现属性所需的实例变量合成出来。这时，需要在分类中为该属性实现`setter`  && `getter`方法，此时，可将存取方法生命为`@dynamic`，就是说，这些方法在运行期再去提供，编译器目前是看不到的。

利用关联对象解决这个问题：

```
#import <objc/runtime.h>

static const char *propertyKey =  "porpertyKey";

@implementation Person

-  (NSArray  *)friends
{
return objc_getAssociatedObject(self,  porpertyKey);
}

-  (void)setFriends:(NSArray  *)friends
{
objc_setAssociatedObject(self,  porpertyKey,  firends,  OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

@end
```

这种方法在技术上可行，但是并不太理想，要把相似的代码写很多遍，而且在内存管理上容易出现错误。例如在为属性实现存取方法时，经常忘记遵从内存管理语义，而且修改属性特质（`attribute`）时，也不能忘记修改存取方法。这里还要考虑可变对象的问题，是否传进一个不可变对象，需要生成一个可变对象的问题。

所以应当把分类理解成一种手段，目的在于扩展类的功能，而非封装数据。

**要点：**

- 把封装数据所用的全部属性都定义在类的主接口里。
- 在“`class-continuation`分类”之外的其他分类中，可以定义存取方法，但是尽量不要定义属性。










