---
layout:     post
title:      理解Objective-C错误类型
subtitle:   《Effective Objective-C 2.0》读书笔记
date:       2018-06-28
author:     Japho
header-img: postimage/post-bg-e2e-ux.jpg
catalog:    true
tags:
    - iOS
---

ARC下默认情况非“异常安全”，如果抛出异常，本应在作用域末尾释放的对象将不会进行自动释放了。若想生成“异常安全”代码，通过设置编译器的标志来实现，不过将引入一些额外代码，在不抛出异常时，也会照样执行这部分代码。这里需要打开的标志叫做  `-fobjc-arc-exceptions`

Objective-C现在所采用的方法是：在极其罕见的情况下抛出异常，异常抛出之后，无需考虑回复问题，而此时应用程序退出。异常只应该应用于极其严重的错误上。在处理“不那么严重的错误”时，Objective-C使用的编程范式为：令方法返回0/nil，或者使用`NSError`。

`NSError`更加灵活，因为经由此对象，可以把导致错误的原因回报给调用者。`NSError`主要封装了三条信息

- Error domain（错误范围，字符串类型）

>错误发生的范围，也就是错误产生的根源，通常是由全局变量来定义，
>
>例如：NSURLErrorDomain

- Error  Code（错误码，整型数）

>独有的错误代码，用于指明在某个范围内发生的错误，通常采用enum

- User Info（用户信息，字典类型）

>有关于该错误的额外信息

设计API时，`NSError`一般是通过委托模式来传递错误，例如：

```
-  (void)connection:(NSURLConnection  *)connection didFailWithError:(NSError  *)error
{
}
```

当发生错误时，会调用此方法进行处理，委托方法不一定需要实现，交给开发人员进行判断。

`NSError`另一种常用方法：经由“输出参数”返回给调用者，

```
-  (BOOL)doSomething:(NSError  **)error
```

传递给方法的参数是一个指针，二该指针又指向另外一个指针，这个指针指向`NSError`对象，也可以把它当成`NSError`对象的指针，如此，该方法不仅能有普通的返回值，还可以经由“输出参数”把`NSError`对象回传给调用者。

```
NSError  *error =  nil;
BOOl  rect =  [obj doSomething:&error];
if (error) 
{
//处理error
}
```

方法会返回布尔值，若关注具体错误，处理经由“输出参数”所返回的错误对象；若不关注，则可以给`error`传入`nil`

```
BOOl  rect =  [obj doSomething:nil];
if (rect)
{
//存在错误，但不获取错误对象
}
```

实际上，在使用ARC时，编译器会把`NSError ** `转换成`NSError *__autoreleasing *`，指针所指向的对象在方法执行完毕之后就会释放掉，所以这个对象必须是自动释放，因为`doSomething:`方法不能保证调用者在方法中创建的`NSError`释放掉，所以这里要加上`autorelease`

**传递“输出参数”**

```
-  (BOOL)doSomething:(NSError  **)error
{
if (/* there was an error*/)
{
if (error)
{
*error = [NSError errorWithDomain:domain code:code userInfo:userInfo];
}
return NO;
}
else
{
return YES;
}
}
```

`*error`为`error`参数“解引用”，使`error`的指针指向新的`NSError`对象，注意：在解引用之前需保证`error`参数不能为`nil`，会因为空指针解引用会导致段错误，使应用崩溃。

错误码一般使用枚举类型。

**要点：**

- 只有发生了可使整个应用程序崩溃的严重错误时，才使用异常。
- 在错误不那么严重的情况下，可以指派委托方法（`delegate`）来处理错误。也可以把错误信息放在`NSError`对象里，经由“输出参数”返回给调用者。












