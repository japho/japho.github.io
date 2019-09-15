---
layout:     post
title:      Objective-C 枚举（位移枚举)
subtitle:   枚举真的有这么简单？快！上车！
date:       2016-11-09
author:     Japho
header-img: postimage/post-bg-debug.png
catalog:    true
tags:
    - iOS
---

## 什么是枚举

在程序设计语言中，一般用一个数值来代表某一状态，这种处理方法不直观，易读性差。如果能在程序中用自然语言中有相应含义的单词来代表某一状态，则程序就很容易阅读和理解。也就是说，事先考虑到某一变量可能取的值，尽量用自然语言中含义清楚的单词来表示它的每一个值，这种方法称为枚举方法，用这种方法定义的类型称枚举类型。

## 枚举的命名

定义的枚举类型名称通常与项目的类文件前缀相同，或者是类库框架缩写,或者跟随具体业务名,如果开头是缩写要大写表示，跟随其后的命名应采用驼峰命名法则，命名应准确表述枚举表示的意义，枚举中各个值都应以定义的枚举类型开头，其后跟随各个枚举值对应的状态、选项或者状态码。

## 状态与选项（states & options）

### 状态
同时只能有一种，如`JFStateError`,`JFStateUnknow`，不可能同时是`JFStateError`和`JFStateUnknow`。如下：

```swift
typedef enum JFState {
    JFStateOK = 0,
    JFStateError,
    JFStateUnknow
} JFState;
```

另外，我们经常在switch语句中使用枚举来表示各个状态，根据各个状态来进项判断。如下

```swift
JFState state = JFStateOK;

    switch (state)
    {
        case JFStateOK:
        {
        }
            break;
        case JFStateError:
        {
        }
            break;
        case JFStateUnknow:
        {
        }
            break;
    }
```

这里大家总是习惯在switch语句中加上default分支，但是在使用枚举来定义状态的时候笔者不建议大家这么做。主要有以下几点：

>- 使用枚举时，所需判断的几种状态是确认可控的，不需要在进行default的判断
>- 在之后的拓展中，如果加入了新的枚举类型，则编译器会发出警告提示开发者switch未处理所有枚举信息，提示新加入的枚举未在switch中进行处理，如果加上default分之的话就不会有该判断

### 选项

定义选项的时候。若这些选项可以彼此组合，则更应如此。只要枚举定义得对，各选项之间就可通过“按位或操作符”（bitwise OR operator）来组合。例如，iOS UI框架中有如下枚举类型，用来表示某个视图应该如何在水平或垂直方向上调整大小。

#### 位移枚举(可复选的枚举) 使用位移实现选项变量

使用枚举定义选项,每个选项均可启用或禁用，使用上述方式来定义枚举值,每个枚举值所对应的二进制表示中，只有1个二进制位的值是1。用“按位或操作符”可组合多个选项。用 `|` 来隔开

***首先来补充下位运算的知识吧 ^_^***

- **1、按位与"&"**

>只有对应的两个二进位均为1时，结果位才为1，否则为0>比如9&5，其实就是1001&0101=0001，因此9&5=1>二进制中，与1相&就保持原位，与0相&就为0 

- **2、按位或"`|`"**
 
>只要对应的二个二进位有一个为1时，结果位就为1，否则为0。>比如9`|`5，其实就是1001`|`0101=1101，因此9`|`5=13

- **3、左移<<**
 
>把整数a的各二进位全部左移n位，高位丢弃，低位补0。左移n位其实就是乘以2的n次方。>例如1<<2 就是0001左移2为0100，因此1<<2=4

**枚举定义如下：**

```swift
typedef NS_OPTIONS(NSUInteger, ActionType) {

    ActionTypeUp    = 1 << 0, //  0001  1
    ActionTypeDown  = 1 << 1, //  0010  2
    ActionTypeRight = 1 << 2, //  0100  4
    ActionTypeLeft  = 1 << 3, //  1000  8

};
```

**枚举判断处理：**

```swift
- (void)action:(ActionType)type
{
    if (type == 0)
    {
        return;
    }

    if ((type & ActionTypeUp) == ActionTypeUp)
    {
        NSLog(@"上");
    }

    if ((type & ActionTypeDown) == ActionTypeDown)
    {
        NSLog(@"下");
    }

    if ((type & ActionTypeLeft) == ActionTypeLeft)
    {
        NSLog(@"左");
    }

    if ((type & ActionTypeRight) == ActionTypeRight)
    {
        NSLog(@"右");
    }

}
```

于是，调用的时候我们通常这么写

```swift
ActionType type = ActionTypeUp | ActionTypeLeft | ActionTypeRight | ActionTypeDown; // 15
[self action:type];
```

- 定义这个`actionType`的选项为四个，这里按位异或，`0001|0010|0100|1000=1111`，得到结果这个type为15。
- 调用方法`[self action:type]`，进行按位与操作：`type & ActionTypeUp`，`1111&0001=0001`，得到这个是否选择了该选项。然后便可以进行判断了。
