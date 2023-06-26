---
layout:     post
title:      AttributeTextParser
subtitle:   Easily build NSAttributedString from XML/HTML like strings.
date:       2023-06-26
author:     Japho
header-img: postimage/post-bg-2015.jpg
catalog: true
tags:
    - iOS
---
## 前言

iOS 开发过程中，经常需要实现一些较为复杂的文本展示，一般需要通过富文本进行处理。iOS 平台下，富文本 API 调用较为繁琐，为了解决这一问题，AttributeTextParser 提供了一套副文本解析方案。

AttributeTextParser 无需关注复杂繁琐的 NSAttributeString 的 API，只需将需要展示的内容拼接为类 HTML 标签，传入即可进行解析。通过类 HTML 标签对需要展示的富文本进行处理，提升代码阅读性、开发效率。

目前 AttributeTextParser 支持已解析类型：**字体**、**颜色**、**图片（静态图片、Gif、序列帧图片）**、**超链**。

## 效果展示

<img src="https://pic.imgdb.cn/item/6492c2e21ddac507ccfb321d.png" width="366" height="292" />

## 语法

- 每个标记需要包含在`<`和`>`之间，类似`<font>`、`<img>`
- 标记中参数格式为 `xxx="xxx"`

#### 字体 \<font\>

```
<font size=\"16.\" color=\"0,1,0,1.\">

```

font 标签以`<font>`为标记，包含以下属性

> **size**：字号（非必须)
> 
> **color**：颜色（非必须），以“,”区分，分别为R、G、B、A，可使用 
> 
> ```+ (NSString *)colorStringWithHexString:(NSString *)color```
> 
> 生成字符串颜色值。该标记之后，到下个<font>标记之前的所有内容中的颜色、字号，均已该标记为准。


**示例**

```
<font size=\"16.\" color=\"0,0,0,1.\">字号16，黑色\n<font size=\"24.\" color=\"1,0,0,1.\">现在为红色，字号24
```

![](https://pic.imgdb.cn/item/6492cbdc1ddac507cc0b9f79.png)

#### 图片 \<img\>

```
<img width=\"40\" height=\"40\" src=\"test.png\"> // 静态图片
<img width=\"40\" height=\"40\" descent=\"0\" src=\"pia\" type=\"1\"> // Gif
<img width=\"40\" height=\"40\" descent=\"0\" src=\"1.png,2.png,3.png\" type=\"2\" fps=\"10\"> // 序列帧
```

img 标签以`<img>`为标记，包含以下属性

>**width**：图片宽度（必须❗️）
>
>**height**：图片宽度（必须❗️）
>
>**scr**：图片地址（必须❗️），可以为本地文件路径、项目本地图片、以“,”分割的序列帧图片
>
>**descent**：图片相对于文字行的间距（非必须），默认为0，descent > 0，图片向下偏移，descent < 0，图片向上便宜，具体偏移量视数值而定
>
>**type**：传递图片的类型（非必须），当图片不是静态图片时，需要传 type 类型。0：静态图片，1：Gif，2：序列帧
>
>**fps**：序列帧图片的fps（非必须），当图片为序列帧时，默认 fps 为6，如果需要修改，可修改此参数

**示例**

```
NSString *text1 = [NSString stringWithFormat:@"<font size=\"16.\" color=\"0,0,0,1.\">字号16，黑色\n
<font size=\"24.\" color=\"1,0,0,1.\">现在为红色，字号24\n\nPng   
<img width=\"200\" height=\"133\" src=\"1.jpeg\">\n\nGif     
<img width=\"250\" height=\"133\" descent=\"0\" src=\"2.gif\" type=\"1\">\n\n序列帧
<img width=\"142\" height=\"34\" descent=\"0\" src=\"%@\" type=\"2\" fps=\"20\">", filePaths];
```

![](https://pic.imgdb.cn/item/6492d5741ddac507cc1aac3f.png)

#### 超链接 \<link\>

```
<link length=\"6\" href=\"www.6.cn\" color=\"0.819608,0.905882,0.713726,1.000000\">链接点击事件
```

link 标签以`<link>`为标记，包含以下属性

>**length**：链接有效长度（必须❗️），`<img>`标签占用1个长度，自该标签后，length 个长度内均响应点击事件，点击事件由代理进行回调。
>
>**href**：点击事件代理返回超链内容（必须❗️），设置点击事件后，点击该区域，代理会返回该 href 用于逻辑处理。
>
>**color**：颜色（非必须），以“,”区分，分别为R、G、B、A，可使用 
> 
> ```+ (NSString *)colorStringWithHexString:(NSString *)color```
> 
> 生成字符串颜色值。该标记之后，到下个`<font>`或`<link>`标记之前的所有内容中的颜色、字号，均已该标记为准。

**示例**

```
<link length=\"6\" href=\"www.6.cn\" color=\"0.819608,0.905882,0.713726,1.000000\">链接点击事件此处非连接点击事件
```

![](https://pic.imgdb.cn/item/649947781ddac507ccfd6841.png)

## AttributeTextParser

#### 核心方法

`attributeStringFromHtmlString: attributes:`

```swift
/// 传入格式化标签，返回适用于 AttributeLabel 的 NSAttributedString
/// @param htmlString 格式化标签
/// @param attributes 附加属性
- (NSAttributedString *)attributeStringFromHtmlString:(NSString *)htmlString
                                           attributes:(nullable NSDictionary<AttributedStringKey, id> *)attributes;

```

该方法将传入的类 HTML 标签转换为`NSAttributedString`，attributes 参数为附件属性，该附加属性将作用于整个富文本字符串中，目前支持字体名称、行间距、断行模式。

```swift
typedef NSString *AttributedStringKey NS_EXTENSIBLE_STRING_ENUM;

OBJC_EXTERN AttributedStringKey const FontAttributedName;
OBJC_EXTERN AttributedStringKey const LineSpacingAttributedName;
OBJC_EXTERN AttributedStringKey const LineBreakModeAttributedName;
```

`getAttributeSizeWithContainerSize: attributeString:`

```swift
/// 获取富文本尺寸
/// @param containerSize 容器尺寸
/// @param attributeString 富文本字符串
- (CGSize)getAttributeSizeWithContainerSize:(CGSize)containerSize attributeString:(NSAttributedString *)attributeString;
```

该方法会返回富文本在 containerSize 下所需占用的尺寸。

#### 点击事件回调

当使用`<link>`处理点击事件时，持有解析器对象的类需遵循以下方法：

```swift
@protocol AttributeTextParserDelegate <NSObject>

@optional

/// 超链接点击回调
/// @param textParser textParser
/// @param href 超链接
- (void)textParser:(AttributeTextParser *)textParser linkTouchedWithHref:(NSString *)href;

@end
```

该方法会将`<link>`标签中的 href 作为参数回调到代理中，以供业务逻辑代码进行处理。

## 使用示例

```swift
- (void)viewDidLoad {
    AttributeTextParser *parser = [[AttributeTextParser alloc] init];
    parser.delegate = self;
    
    NSString *htmlString = @"<font size=\"16.\" color=\"0,0,0,1.\">字号16，黑色";
    NSAttributedString *attibuteStr = [parser attributeStringFromHtmlString:htmlString attributes:@{
        FontAttributedName : @"PingFangSC-Semibold",
        LineSpacingAttributedName : @(20),
        LineBreakModeAttributedName : @(NSLineBreakByTruncatingTail)
    }];
    
    AttributedLabel *label = [[AttributedLabel alloc] init];
    label.attributedText = attibuteStr;
    [self.view addSubview:label];
}

#pragma mark - AttributeTextParserDelegate
- (void)textParser:(AttributeTextParser *)textParser linkTouchedWithHref:(NSString *)href {
    NSLog(@"Href : %@", href);
}

```

- 初始化 AttributeTextParser，如果需要点击事件响应，则需添加代理。

```swift
AttributeTextParser *parser = [[AttributeTextParser alloc] init];
parser.delegate = self;
```

- 添加代理回调

```swift
#pragma mark - AttributeTextParserDelegate
- (void)textParser:(AttributeTextParser *)textParser linkTouchedWithHref:(NSString *)href {
    NSLog(@"Href : %@", href);
}
```

- 将拼接好的类 HTML 标签生成 NSAttributedString

```swift
NSAttributedString *attibuteStr = [parser attributeStringFromHtmlString:htmlString attributes:@{
	FontAttributedName : @"PingFangSC-Semibold",
	LineSpacingAttributedName : @(20),
	LineBreakModeAttributedName : @(NSLineBreakByTruncatingTail)
}];
```

这里可以添加附属属性 attributes，以字典的形式传递，分别为：

> FontAttributedName：字体名称
>
> LineSpacingAttributedName：行间距
>
> LineBreakModeAttributedName：断行模式

- 调用 AttributedLabel 或 AttributedTextView 进行展示

```swift
AttributedLabel *label = [[AttributedLabel alloc] init];
label.attributedText = attibuteStr;
[self.view addSubview:label];
```

## 源码解析

```swift

- (NSAttributedString *)attributeStringFromHtmlString:(NSString *)htmlString attributes:(NSDictionary *)attributes {
    NSMutableAttributedString *finalString = [[NSMutableAttributedString alloc] initWithString:htmlString];
    // 通过正则将类 HTML 标签拆分
    NSRegularExpression *regex = [NSRegularExpression regularExpressionWithPattern:@"<[^>]+>" options:NSRegularExpressionCaseInsensitive error:nil];
    NSArray *resultArray = [regex matchesInString:htmlString options:0 range:NSMakeRange(0, htmlString.length)];
    for (NSTextCheckingResult *result in resultArray) {
        // 获取每个 <> 标签，e.g.:<font size="14.000000">
        NSString *htmlAttributedString = [htmlString substringWithRange:result.range];
        // 去除 <>，font size="14.000000"
        NSString *htmlAttributedContent = [htmlAttributedString substringWithRange:NSMakeRange(1, htmlAttributedString.length - 2)];
        
        // font
        // 获取每个 <font> 标签中字体属性
        NSDictionary *fontInfo = [self fontInfoWithString:htmlAttributedContent attributes:attributes];
        if (fontInfo != nil) {
            // fontInfo 存在，则表示本次 forin 循环遍历为 <font> 标签，定位标签位置
            NSRange fontInfoRange = [finalString.string rangeOfString:htmlAttributedString options:NSLiteralSearch];
            // 移除标记语言 <font>
            [finalString replaceCharactersInRange:fontInfoRange withString:@""];
            // 字体设置范围为自此 <font> 标签后的所有字符串
            NSRange replaceRange = NSMakeRange(fontInfoRange.location, finalString.length - fontInfoRange.location);
            // 设置范围内字体
            if ([[fontInfo allKeys] containsObject:NSFontAttributeName]) {
                UIFont *font = [fontInfo objectForKey:NSFontAttributeName];
                [finalString yy_setFont:font range:replaceRange];
            }
            // 设置范围内颜色
            if ([[fontInfo allKeys] containsObject:NSForegroundColorAttributeName]) {
                UIColor *color = [fontInfo objectForKey:NSForegroundColorAttributeName];
                [finalString yy_setColor:color range:replaceRange];
            }
        }
        
        // image
        // 获取每个 <img> 标签中属性
        NSDictionary *imageInfo = [self imageInfoWithString:htmlAttributedContent attributes:attributes];
        if (imageInfo != nil) {
            // imageInfo 存在，则表示本次 forin 循环遍历为 <img> 标签
            CGFloat width = [[imageInfo objectForKey:@"width"] floatValue];
            CGFloat height = [[imageInfo objectForKey:@"height"] floatValue];
            CGFloat descent = [[imageInfo objectForKey:@"descent"] floatValue]; // 向下偏移量
            NSString *fileName = [imageInfo objectForKey:@"fileName"];
            CGFloat fps = [[imageInfo objectForKey:@"fps"] floatValue];
            CGFloat duration = [[imageInfo objectForKey:@"duration"] floatValue];
            EnumAttributeTextImageType imageType = [[imageInfo objectForKey:@"type"] integerValue];
            
            // 生成图片 attachment
            id attrachmentContent;
            switch (imageType) {
                case EnumAttributeTextImageTypeDefault: { // 静态图片
                    YYImage *image = [self imageWithFilePath:fileName];
                    attrachmentContent = image;
                    break;
                }
                case EnumAttributeTextImageTypeGif: { // 动图 Gif
                    YYImage *image = [YYImage imageNamed:fileName];
                    image.preloadAllAnimatedImageFrames = YES;
                    attrachmentContent = image;
                    break;
                }
                case EnumAttributeTextImageTypeFrame: { // 序列帧
                    NSArray *imageStringArray = [fileName componentsSeparatedByString:@","];
                    // 默认 fps 为 6
                    NSTimeInterval defaultDuration = 1.f / 6.f; // defaultDuration 为每帧时间
                    if (duration != 0 && imageStringArray.count != 0) {
                        defaultDuration = duration / imageStringArray.count; // 动画总时间 / 动画总帧数 = 每帧所用时间
                    }
                    if (fps != 0) {
                        defaultDuration = 1.f / fps;
                    }
                    YYFrameImage *frameImage = [[YYFrameImage alloc] initWithImagePaths:imageStringArray
                                                                       oneFrameDuration:defaultDuration
                                                                              loopCount:0];
                    attrachmentContent = frameImage;
                    break;
                }
            }
            
            NSMutableAttributedString *attachString = [NSMutableAttributedString v6cn_attachmentStringWithImage:attrachmentContent
                                                                                                         ascent:height - descent
                                                                                                        descent:descent
                                                                                                           size:CGSizeMake(width, height)];
            if (attachString) {
                // attachString 存在，替换 <img> 标签
                NSRange imageInfoRange = [finalString.string rangeOfString:htmlAttributedString options:NSLiteralSearch];
                [finalString replaceCharactersInRange:imageInfoRange withAttributedString:attachString];
            }
        }
    }
    
    NSMutableDictionary *linkHrefDictionaryM = [NSMutableDictionary dictionary];
    // 处理完除 link 的其他标签之后，再去处理 link
    for (NSTextCheckingResult *result in resultArray) {
        // 获取每个 <> 标签，e.g.: <link length="4" href="www.6.cn" color="0.819608,0.905882,0.713726,1.000000">
        NSString *htmlAttributedString = [htmlString substringWithRange:result.range];
        // 去除 <>，link length="4" href="www.6.cn" color="0.819608,0.905882,0.713726,1.000000"
        NSString *htmlAttributedContent = [htmlAttributedString substringWithRange:NSMakeRange(1, htmlAttributedString.length - 2)];
        
        //link
        // 获取每个 <link> 标签中属性
        NSDictionary *linkInfoDictionary = [self linkInfoWithString:htmlAttributedContent attributes:attributes];
        if (linkInfoDictionary != nil) {
            NSInteger linkLength = [[linkInfoDictionary objectForKey:@"length"] integerValue];
            NSString *href = [linkInfoDictionary objectForKey:@"href"];
            NSDictionary *linkAttributes = [linkInfoDictionary objectForKey:@"attributes"];
            
            // link 标记符 range，<link length="4" href="www.6.cn" color="0.819608,0.905882,0.713726,1.000000">，<>
            NSRange linkInfoRange = [finalString.string rangeOfString:htmlAttributedString options:NSLiteralSearch];
            
            // 防止 location 越界
            if (linkInfoRange.location > finalString.length - 1) {
                linkInfoRange.location = finalString.length - 1;
            }
            
            // 删除 <link> 标签
            [finalString replaceCharactersInRange:linkInfoRange withString:@""];
            
            if (finalString.length < linkInfoRange.location + linkLength) {
                linkLength = finalString.length - linkInfoRange.location;
            }
            
            if (linkLength < 0) {
                linkLength = 0;
            }
            
            // 超链range，超链之后开始计算，受 length 控制，图片长度计算为 1
            NSRange linkRange = NSMakeRange(linkInfoRange.location, linkLength);
            
            // 用于储存 href，以 range 为 key
            if (href && href.length) {
                [linkHrefDictionaryM setValue:href forKey:NSStringFromRange(linkRange)];
            }
            
            UIColor *highlightColor;
            if (linkAttributes && linkAttributes.allKeys.count) {
                if ([[linkAttributes allKeys] containsObject:NSForegroundColorAttributeName]) {
                    highlightColor = [linkAttributes objectForKey:NSForegroundColorAttributeName];
                }
            }
            
            [finalString yy_setTextHighlightRange:linkRange
                                            color:highlightColor
                                  backgroundColor:[UIColor clearColor]
                                        tapAction:^(UIView * _Nonnull containerView, NSAttributedString * _Nonnull text, NSRange range, CGRect rect) {
                NSArray *linkHrefKeysArray = [linkHrefDictionaryM allKeys];
                for (NSString *tempKey in linkHrefKeysArray) {
                    if ([tempKey isEqualToString:NSStringFromRange(range)]) {
                        NSString *href = [linkHrefDictionaryM objectForKey:tempKey];
                        if (self.delegate && [self.delegate respondsToSelector:@selector(textParser:linkTouchedWithHref:)]) {
                            [self.delegate textParser:self linkTouchedWithHref:href];
                        }
                    }
                }
            }];
        }
    }
    
    // 去除其他 HTML 标签
    NSArray *arrHtmlTagResult = [regex matchesInString:finalString.string options:0 range:NSMakeRange(0, finalString.length)];
    for (NSInteger i = arrHtmlTagResult.count - 1; i >= 0; i--) {
        NSTextCheckingResult *result = arrHtmlTagResult[i];
        [finalString replaceCharactersInRange:result.range withString:@""];
    }
    
    // 设置追加属性
    CGFloat lineSpacing = [[attributes objectForKey:LineSpacingAttributedName] floatValue];
    NSLineBreakMode lineBreakMode = [[attributes objectForKey:LineBreakModeAttributedName] integerValue];
    
    finalString.yy_lineSpacing = lineSpacing;
    finalString.yy_lineBreakMode = lineBreakMode;
    finalString.yy_baseWritingDirection = NSWritingDirectionLeftToRight;
    finalString.yy_writingDirection = @[@(NSWritingDirectionLeftToRight | NSWritingDirectionOverride)];
    
    return finalString;
}

```

## 标签拓展

当现有标签不满足需求时，可添加拓展标签，类似
> `<b>`加粗
> 
> `<i>`斜体
> 
> `<badge id=56>`徽章视图
> 
> `<fans name="张三">`粉丝团徽章
> 
> `<view id=0xdddr3333>`插入UIView

只需在`attributeStringFromHtmlString: attributes:`中继续拓展其他类型即可实现。

## 关于第三方依赖

目前项目中 AttributeTextParser 集成了 YYKit，通过 YYKit 来实现富文本相关内容，但 AttributeTextParser 不与富文本第三方强耦合，可以使用 NSAttributeString 原生 API 进行实现，也可使用其他富文本第三方进行实现；

**关于原生方法实现思路**

>- 通过 `addAttributes: range:` 方法可实现类似字体 NSFontAttributeName、颜色 NSForegroundColorAttributeName、超链 NSLinkAttributeName 等功能；
>- 通过 NSTextAttachment 添加图片，虽然 NSTextAttachment 默认不支持动图，但是可以通过复写`NSTextAttachmentContainer`类中`imageForBounds: textContainer: characterIndex:`方法进行实现
>- 关于点击事件，可以为 `UITextView` 添加 tap 事件，可通过下述方法获取 NSLinkAttributeName 的值，通过 `UITextView` 回调给外部，进行事件的传递。
 
 ```
UITextPosition *position = [self.textView closestPositionToPoint:[tap locationInView:self.textView]];
NSDictionary<NSAttributedStringKey, id> *textStylings = [self.textView textStylingAtPosition:position inDirection:UITextStorageDirectionForward];
NSString *url = [textStylings objectForKey:NSLinkAttributeName];
 ```

## One More Thing

在现有架构下，房间公聊解析可进一步容器化，去除本地 socket 解析逻辑，将业务逻辑交给前端、服务端解析，返回一段类 HTML 标签，端上公聊展示，对于单一符文本样式（无定制UI需求），只做展示解析处理，不再处理业务。
 