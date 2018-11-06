---
layout:     post
title:      如何查找当前的第一响应者 
subtitle:   不调用私有API查找第一响应者
date:       2017-02-14
author:     Japho
header-img: postimage/post-bg-miui6.jpg
catalog:    true
tags:
    - iOS
---

有时候总是有需求来获取当前的第一响应者，例如让TextField收键盘，隐藏视图等等操作都需要获取当前的第一响应者，那么该如何获取呢？

```swift
UIWindow *keyWindow = [[UIApplication sharedApplication] keyWindow];  
UIView *firstResponder = [keyWindow performSelector:@selector(firstResponder)];  
  
NSLog(@"%@",firstResponder);
```

`注意：`这个方法虽然简单，但是调用了私有API在平时调试的时候可以使用这种方法，但是打包上线的时候需要把该方法屏蔽掉，不然极有可能被打回。

下面介绍下现在公认比较好的一种方法：

建立分类`UIResponder+FirstResponder`

`UIResponder+FirstResponder.h`


```swift
//  
//  UIResponder+FirstResponder.h  
//  BobcareDoctorApp  
//  
//  Created by Japho on 16/3/23.  
//  Copyright © 2016年 com.01wisdom. All rights reserved.  
//  
  
#import <UIKit/UIKit.h>  
  
@interface UIResponder (FirstResponder)  
  
+ (id)currentFirstResponder;  
  
@end
```

`UIResponder+FirstResponder.m`


```swift
//  
//  UIResponder+FirstResponder.m  
//  BobcareDoctorApp  
//  
//  Created by Japho on 16/3/23.  
//  Copyright © 2016年 com.01wisdom. All rights reserved.  
//  
  
#import "UIResponder+FirstResponder.h"  
  
static __weak id currentFirstResponder;  
  
@implementation UIResponder (FirstResponder)  
  
+ (id)currentFirstResponder {  
    currentFirstResponder = nil;  
    [[UIApplication sharedApplication] sendAction:@selector(findFirstResponder:) to:nil from:nil forEvent:nil];  
    return currentFirstResponder;  
}  
  
- (void)findFirstResponder:(id)sender {  
    currentFirstResponder = self;  
}  
  
@end
```