---
layout:     post
title:      iOS UILabel垂直居中
subtitle:   解决UILabel默认顶格显示问题
date:       2016-09-15
author:     Japho
header-img: postimage/post-bg-universe.jpg
catalog:    true
tags:
    - iOS
---

平时开发的时候可能会遇到这种问题：当一个UILabel的frame的高度设置的过大时，发现UILabel是垂直居中的，有的需求是需要将这个Label垂直向上显示，之前的办法是计算出label.text的字体所占用的frame大小，根据这个大小再重新设置label的frame值，未免有些麻烦，前阵子封装了个自定义label实现的垂直居中的设置。废话不多说，上代码。

`JFLabel.h`

```swift
//  
//  JFLabel.h  
//  BobcareDoctorApp  
//  
//  Created by Japho on 16/2/25.  
//  Copyright © 2016年 com.01wisdom. All rights reserved.  
//  
  
#import <UIKit/UIKit.h>  
  
typedef enum  
{  
    VerticalAlignmentTop = 0, // default  
    VerticalAlignmentMiddle,  
    VerticalAlignmentBottom,  
} VerticalAlignment;  
  
@interface JFLabel : UILabel  
{  
      
@private  
      
    VerticalAlignment _verticalAlignment;  
}  
  
@property (nonatomic) VerticalAlignment verticalAlignment;  
  
@end  
```

`JFLabel.m`

```swift
//  
//  JFLabel.m  
//  BobcareDoctorApp  
//  
//  Created by Japho on 16/2/25.  
//  Copyright © 2016年 com.01wisdom. All rights reserved.  
//  
  
#import "JFLabel.h"  
  
@implementation JFLabel  
  
@synthesize verticalAlignment = verticalAlignment_;   
  
- (id)initWithFrame:(CGRect)frame {  
    if (self = [super initWithFrame:frame]) {  
        self.verticalAlignment = VerticalAlignmentMiddle;  
    }  
    return self;  
}  
  
- (void)setVerticalAlignment:(VerticalAlignment)verticalAlignment {  
    verticalAlignment_ = verticalAlignment;  
    [self setNeedsDisplay];  
}  
  
- (CGRect)textRectForBounds:(CGRect)bounds limitedToNumberOfLines:(NSInteger)numberOfLines {  
    CGRect textRect = [super textRectForBounds:bounds limitedToNumberOfLines:numberOfLines];  
    switch (self.verticalAlignment) {  
        case VerticalAlignmentTop:  
            textRect.origin.y = bounds.origin.y;  
            break;  
        case VerticalAlignmentBottom:  
            textRect.origin.y = bounds.origin.y + bounds.size.height - textRect.size.height;  
            break;  
        case VerticalAlignmentMiddle:  
            // Fall through.  
        default:  
            textRect.origin.y = bounds.origin.y + (bounds.size.height - textRect.size.height) / 2.0;  
    }  
    return textRect;  
}  
  
-(void)drawTextInRect:(CGRect)requestedRect {  
    CGRect actualRect = [self textRectForBounds:requestedRect limitedToNumberOfLines:self.numberOfLines];  
    [super drawTextInRect:actualRect];  
}  
  
@end
```

封装的类继承自UILabel，需要设置垂直居中时直接设置属性就可以了。
调用示例代码：


```swift
- (JFLabel *)titleLabel  
{  
    if (!_titleLabel)  
    {  
        _titleLabel = [[JFLabel alloc] initWithFrame:CGRectMake(15, 15, SCREEN_WIDTH - CASE_IMAGE_VIEW_WIDTH - 15 - 20 - 5, 40)];  
        _titleLabel.text = @"测试垂直居中文字";  
        _titleLabel.font = [UIFont systemFontOfSize:16];  
        _titleLabel.numberOfLines = 0;  
        _titleLabel.textColor = [UIColor blackColor];  
        _titleLabel.verticalAlignment = VerticalAlignmentTop;//垂直居中  
    }  
      
    return _titleLabel;  
}
```