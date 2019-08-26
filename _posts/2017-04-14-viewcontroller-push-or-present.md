---
layout:     post
title:      判断当前viewcontroller是push还是present的方式显示
subtitle:   妈妈再也不用担心我不会判断了
date:       2017-04-14
author:     Japho
header-img: postimage/post-bg-iWatch.jpg
catalog:    true
tags:
    - iOS
---

通过`presentviewcontroller`的方式显示的`viewcontroller`不会存入`self.navigationController.viewControllers`数组中。而通过push方式显示的`viewcontroller`会存在该数组的最后。

代码如下：

```swift
NSArray *viewcontrollers = self.navigationController.viewControllers;    
    
if (viewcontrollers.count > 1)     
{    
    if ([viewcontrollers objectAtIndex:viewcontrollers.count-1]==self)     
    {    
        //push方式    
        [self.navigationController popViewControllerAnimated:YES];    
    }    
}    
else    
{    
    //present方式    
    [self.navigationController dismissViewControllerAnimated:YES completion:nil];    
}
```
