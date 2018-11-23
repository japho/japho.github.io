---
layout:     post
title:      编码习惯问题
subtitle:   《Effective Objective-C 2.0》读书笔记
date:       2018-06-29
author:     Japho
header-img: https://ws4.sinaimg.cn/large/006tNbRwly1fxi1iri19sj31gs0o6myu.jpg
catalog:    true
tags:
    - iOS
    - 读书笔记
---

## 将类的实现代码分散到便于管理的数个分类中

当类中方法及实现过多的时候，可以考虑用Objective-C的“分类”机制，把代码按逻辑划分到几个分区中。通过分类机制，可以把类代码分成很多个易于管理的小块，以便单独检视。此外，使用分类还有一个原因，就是便于调试。对于某个分类中的所有方法来说，分类名称都会出现在其符号中，例如

```
- [Person(FriendShip)  addFriend:]
```

编写程序库时，可以考虑创建`private`分类，经常会遇到一些方法，这些方法不是公共的API，但却非常适合在程序库内部使用，此时应该创建爱你`private`分类，如果程序中某些地方要用到这些方法，就引入这个分类文件。二分类的头文件不随程序库一并公开，于是，该代码库的使用者不会知晓库里还有这些私有方法。

**要点：**

- 使用分类机制把勒种的实现代码划分成易于管理的小块。
- 将应该视为“私有”的方法归入private分类中，以隐藏实现细节。

## 总是为第三方类的分类名称加前缀

分类机制通常用于向无源码的既有类中添加新功能，但在使用时会存在问题：分类中的方法是直接添加在类中的，将分类方法加入类中这一操作是在运行期系统加载分类时完成的。运行时系统会把分类中所实现的每个方法都加入类的方法列表中，如果类中本来就存在这个方法，那么就会产生覆盖，若发生多次覆盖，则以最后一次为准。

例如

```
@interface NSString  (ABC_HTTP)

- (NSString  *)abc_urlEncodedString;

- (NSString  *)abc_urlDecodedString;
```

**要点：**

- 向第三方类中添加分类时，总应给其名称加上专用前缀。
- 向第三方类中添加分类时，总应给其的方法加上专用前缀。

## 使用“class-continuation分类”隐藏实现细节

**要点：**

- 通过`class-continuation`分类向类中新增实例变量。
- 如果某属性在主接口中声明为“只读”，而类内部又要用`setter`方法修改次数性，那么就在“`class-continuation`分类”中将其扩展为“可读写”。
- 把私有方法的原型声明在“`class-continuation`分类”中。
- 若想使用类所遵循的协议而不为人所知，则可于“`class-continuation`分类”中声明。

## 用“僵尸对象”调试内存管理问题

**要点：**

- 系统在回收对象时，可以不将其真正的回收，而是把它转化为僵尸对象。通过环境变量`NSZombieEnabled`可开启此功能。
- 系统会修改对象的`isa`指针，令其指向特殊的僵尸类，从而使该对象变为僵尸对象。僵尸类能够响应所有的选择子，响应方式为：打印一条包含消息内容及其接受者的消息，然后终止应用程序。

## 不要使用retainCount

ARC已经将此方法废弃了，在ARC中调用此方法，编译器会报错。

此方法无用原因：他返回的引用计数只是某个给定时间点上的值，该方法并未考虑到系统会稍后吧自动释放池清空，因为不会将后续的释放操作从返回值里减去。所以这个值未必能真实的反映实际的保留计数。
我们不应该总是依赖保留计数的具体值来编码。即便是只为了调试，此方法也不是很有用。由于对象可能处在自动释放池中，所以其保留计数未必如想象中精确。而且其他程序库也有可能自行保留或释放对象，这都会扰乱保留计数的具体取值。

**要点：**

- 对象的保留计数看似有用，实则不然，因为任何给定时间点上的“绝对保留计数”都无法反映对象生命期的全貌。
- 引入ARC之后，retainCount方法就正式废止了，在ARC下调用该方法会导致编译器报错。

## 为常用的块类型创建typedef

```
return_type (^block_name)(parameters)
```

为了隐藏复杂的块类型，需要用到C语言中名为类型定义的特性。例如：

```
typedef int (^EOCSomeBlock)(BOOL  flag,  int value);
```

声明变量时，把名称放在类型中间，并在前面加上“^”符号。以下来创建变量，直接使用新类型即可：

```
EOCSomeBlock  block =  ^(BOOL  flag,  int value){
//Impementation
}
```

**要点：**

- 以`typedef`重新定义块类型，可令块变量用起来更加简单。
- 定义新类型时应遵从现有的命名习惯，勿使其名称与别的类型冲突。
- 不妨为同一块签名定义多个类型别名。如果需要重构的代码使用了块类型的某个别名，那么只需修改相应`typedef`中的块签名即可，无需改动其他`typedef`。

## 用handler块来降低代码分散程度

编写代码时，经常会遇到一种范式，就是“异步执行任务”。异步方法执行完任务后，需要以某种手段通知代码。实现这种功能有很多方法，常用技巧是委托协议令关注此事件的对象遵从该协议。对象成为`delegate`之后，就可以在相关事件发生时得到通知了。

```
#import <Foundation/Foundation.h>

@class EOCNetWorkFetcher;
@protocol EOCNetWorkFetcherDelegate <NSObject>
- (void)netWorkFetcher:(EOCNetWorkFetcher *)fetcher didFinishWithData:(NSData *)data;
@end

@interface EOCNetWorkFetcher : NSObject
@property (nonatomic, weak) id<EOCNetWorkFetcherDelegate> delegate;
- (void)initWithUrl:(NSURL *)url;
- (void)start;
@end;
```

其他类调用：

```
- (void)fetchFooData{
NSURL *url = [[NSURL alloc] initWithString:@"http://www.xvideos.com"];
EOCNetWorkFetcher *fetcher = [[EOCNetWorkFetcher alloc] initWithURL:url];
fetcher.delegate = self;
[fetcher start];
}

- (void)netWorkFetcher:(EOCNetWorkFetcher *)fetcher didFinishWithData:(NSData *)data
{
_fetcherFooData = data;
}
```
