---
layout:     post
title:      iOS 自定义常见第三方分享
subtitle:   基于原生SDK开发实现
date:       2016-09-30
author:     Japho
header-img: postimage/post-bg-miui-ux.jpg
catalog:    true
tags:
    - iOS
---

## 前言

平时经常会遇到做第三分享的需求，相信大家好多都使用的集成的分享平台shareSDK、友盟分享等。他们其实是对各种第三方平台进行了二次封装，有时需求只要求做其中一种平台的分享的时候其实像shareSDK这种集成环境就有些显得比较冗余了，因为他集成了很多的平台，用户可自定义程度不高，使得自己的程序会变得很臃肿，今天就把自己的干货拿出来，来讲一讲自定义的第三方分享。
下面主要讲一下目前应用比较广的，QQ、微信、微博的分享。

## 集成SDK

首先要集成你要做分享平台的SDK，和相应环境的配置，这些配置在我之前的博客中都有所提及，我就不在这里再次叙述了

- 微博SDK的集成：[微博SDK集成](http://www.jianshu.com/p/c166d424866e)
- QQ SDK集成：[QQ SDK集成](http://www.jianshu.com/p/567d39aeac80)
- 微信SDK集成：[微信SDK集成](http://www.jianshu.com/p/92902fecb861)

注意在集成SDK后需要在`Appdelegate`中进行注册

## 微博的分享

```swift
+ (void)weiboShareLinkWithUrl:(NSString *)url title:(NSString *)title description:(NSString *)description andPicUrl:(NSString *)picUrl  
{  
    NSLog(@"调用了微博");  
      
    AppDelegate *myDelegate =(AppDelegate*)[[UIApplication sharedApplication] delegate];  
      
    WBAuthorizeRequest *authRequest = [WBAuthorizeRequest request];  
    authRequest.redirectURI = @"这里填写你申请微博应用时的回调网址";  
    authRequest.scope = @"all";  
      
    WBMessageObject *message = [WBMessageObject message];  
      
    //设置分享的标题  
    message.text = title;  
      
    WBWebpageObject *webpage = [WBWebpageObject object];  
    webpage.objectID = @"identifier1";  
    webpage.title = title;  
    webpage.description = description;  
      
    //设置分享出去显示的图片  
    if (picUrl.length == 0 || picUrl == nil)  
    {  
        webpage.thumbnailData = [NSData dataWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"icon_share" ofType:@"png"]];  
    }  
    else  
    {  
        webpage.thumbnailData = [NSData dataWithContentsOfURL:[NSURL URLWithString:picUrl]];  
    }  
    webpage.webpageUrl = url;  
    message.mediaObject = webpage;  
      
    WBSendMessageToWeiboRequest *request = [WBSendMessageToWeiboRequest requestWithMessage:message authInfo:authRequest access_token:myDelegate.wbtoken];  
    request.userInfo = @{@"ShareMessageFrom": @"SendMessageToWeiboViewController",  
                         @"Other_Info_1": [NSNumber numberWithInt:123],  
                         @"Other_Info_2": @[@"obj1", @"obj2"],  
                         @"Other_Info_3": @{@"key1": @"obj1", @"key2": @"obj2"}};  
      
    //调用此方法进行分享  
    [WeiboSDK sendRequest:request];  
}
```

## QQ的分享

QQ的分享分为分享给QQ好友和分享到QQ空间，这里首先定义一个枚举类型来区分是哪一种分享

在分享工具类的头文件里：

`JFTools.h`

```swift
typedef enum : NSUInteger {  
    QQShare_Friends,  
    QQShare_Qzone,  
} QQShare_Type;  
```

然后实现分享方法： 

```swift
+ (void)qqShareLinkWithScene:(QQShare_Type)scene url:(NSString *)url title:(NSString *)title andDescription:(NSString *)description  
{  
    NSURL *shareUrl = [NSURL URLWithString:url];  
      
    NSLog(@"调用了QQ");  
      
    NSString *picUrl = @"这里填写你要分享出图的图片链接";  
    NSURL *imageUrl = [NSURL URLWithString:picUrl];  
   
    //通过QQAPIObject来创建将要分享的对象     
    QQApiObject *newsObj = [QQApiNewsObject objectWithURL:shareUrl title:title description:description previewImageURL:imageUrl targetContentType:QQApiURLTargetTypeNews];  
    //设置分享Req对象  
    SendMessageToQQReq *req = [SendMessageToQQReq reqWithContent:newsObj];  
      
    switch (scene)  
    {  
            //分享给QQ好友  
        case QQShare_Friends:  
        {  
            [QQApiInterface sendReq:req];  
            break;  
        }  
            //分享给QQ空间  
        case QQShare_Qzone:  
        {  
            [QQApiInterface SendReqToQZone:req];  
            break;  
        }  
        default:  
            break;  
    }  
}
```

## 微信的分享

微信的分享也同QQ一样，是分为分享微信好友和分享微信朋友圈，这里只要引入微信的头文件就可以了，微信官方SDK已经声明了所需要用的枚举了，无需我们再进行设置了。

用以下代码实现分享：

```swift
+ (void)wechatShareLinkWithScene:(int)scene url:(NSString *)url title:(NSString *)title andDescription:(NSString *)description  
{  
    //封装mediaMessage对象  
    WXMediaMessage *message = [WXMediaMessage message];  
      
    //设置微信分享的title  
    [message setTitle:title];  
    //设置分享描述内容  
    [message setDescription:description];  
    //设置分享所需图片  
    [message setThumbImage:[UIImage imageNamed:@"icon_share"]];  
      
    NSLog(@"调用了微信");  
      
    //封装WXWebpageObject对象  
    WXWebpageObject *ext = [WXWebpageObject object];  
    ext.webpageUrl = url;  
      
    message.mediaObject = ext;  
      
    //发送请求  
    SendMessageToWXReq* req = [[SendMessageToWXReq alloc] init];  
    req.bText = NO;  
    req.message = message;  
    req.scene = scene;  
      
    [WXApi sendReq:req];  
}
```

这样就实现了简单的分享，一般的需求几乎都可以满足，是不是很简单。

## 总结下

首先集成你准备分享平台的SDK，然后进行Appdelegate的注册，然后实现分享，这里需要注意下，微信和微博分享的图片是传本地图片，QQ是要传图片的链接，(这里要吐槽下，不理解传链接有什么用)。

既然分享就要分享的彻底点，下面po出博主自己封装的分享工具类，如有不足之处还请大家多提意见😏

`JFShareTool.h`

```swift
//  
//  JFShareTool.h  
//  BobcareDoctorApp  
//  
//  Created by wangjifeng on 15/11/13.  
//  Copyright © 2015年 com.01wisdom. All rights reserved.  
//  
  
#import <Foundation/Foundation.h>  
  
typedef enum : NSUInteger {  
    QQShare_Friends,  
    QQShare_Qzone,  
} QQShare_Type;  
  
@interface JFShareTool : NSObject  
  
/** 
*  分享链接给微信 
* 
*  @param scene       场景 
*  @param url         url 
*  @param description 描述 
*/  
+ (void)wechatShareLinkWithScene:(int)scene url:(NSString *)url title:(NSString *)title andDescription:(NSString *)description;  
  
/** 
 *  分享链接给QQ 
 * 
 *  @param scene 场景 
 *  @param url   url 
 *  @param description 描述 
 */  
+ (void)qqShareLinkWithScene:(QQShare_Type)scene url:(NSString *)url title:(NSString *)title andDescription:(NSString *)description;  
  
/** 
 *  分享链接给微博 
 * 
 *  @param url         链接 
 *  @param description 描述 
 */  
+ (void)weiboShareLinkWithUrl:(NSString *)url title:(NSString *)title description:(NSString *)description andPicUrl:(NSString *)picUrl;  
  
@end
```

`JFShareTool.m`

```swift
//  
//  JFShareTool.m  
//  BobcareDoctorApp  
//  
//  Created by wangjifeng on 15/11/13.  
//  Copyright © 2015年 com.01wisdom. All rights reserved.  
//  
  
#import "JFShareTool.h"  
#import "WXApi.h"  
#import <UIKit/UIKit.h>  
#import <TencentOpenAPI/QQApiInterface.h>  
#import "WeiboSDK.h"  
#import "AppDelegate.h"  
  
@implementation JFShareTool  
  
+ (void)wechatShareLinkWithScene:(int)scene url:(NSString *)url title:(NSString *)title andDescription:(NSString *)description  
{  
    //封装mediaMessage对象  
    WXMediaMessage *message = [WXMediaMessage message];  
      
    [message setTitle:title];  
    [message setDescription:description];  
    [message setThumbImage:[UIImage imageNamed:@"icon_share"]];  
      
    NSLog(@"调用了微信");  
      
    //封装WXWebpageObject对象  
    WXWebpageObject *ext = [WXWebpageObject object];  
    ext.webpageUrl = url;  
      
    message.mediaObject = ext;  
      
    //发送请求  
    SendMessageToWXReq* req = [[SendMessageToWXReq alloc] init];  
    req.bText = NO;  
    req.message = message;  
    req.scene = scene;  
      
    [WXApi sendReq:req];  
}  
  
+ (void)qqShareLinkWithScene:(QQShare_Type)scene url:(NSString *)url title:(NSString *)title andDescription:(NSString *)description  
{  
    NSURL *shareUrl = [NSURL URLWithString:url];  
      
    NSLog(@"调用了QQ");  
      
    NSString *picUrl = @"http://file.bmob.cn/M02/B3/F9/oYYBAFZSehaAdoeSAABBZ_Q9h-U696.png";  
    NSURL *imageUrl = [NSURL URLWithString:picUrl];  
      
    QQApiObject *newsObj = [QQApiNewsObject objectWithURL:shareUrl title:title description:description previewImageURL:imageUrl targetContentType:QQApiURLTargetTypeNews];  
    SendMessageToQQReq *req = [SendMessageToQQReq reqWithContent:newsObj];  
      
    switch (scene)  
    {  
            //分享给QQ好友  
        case QQShare_Friends:  
        {  
            [QQApiInterface sendReq:req];  
            break;  
        }  
            //分享给QQ空间  
        case QQShare_Qzone:  
        {  
            [QQApiInterface SendReqToQZone:req];  
            break;  
        }  
        default:  
            break;  
    }  
}  
  
+ (void)weiboShareLinkWithUrl:(NSString *)url title:(NSString *)title description:(NSString *)description andPicUrl:(NSString *)picUrl  
{  
    NSLog(@"调用了微博");  
      
    AppDelegate *myDelegate =(AppDelegate*)[[UIApplication sharedApplication] delegate];  
      
    WBAuthorizeRequest *authRequest = [WBAuthorizeRequest request];  
    authRequest.redirectURI = @"http://www.bobcare.com";  
    authRequest.scope = @"all";  
      
    WBMessageObject *message = [WBMessageObject message];  
      
    message.text = title;  
      
    WBWebpageObject *webpage = [WBWebpageObject object];  
    webpage.objectID = @"identifier1";  
    webpage.title = title;  
    webpage.description = description;  
      
    if (picUrl.length == 0 || picUrl == nil)  
    {  
        webpage.thumbnailData = [NSData dataWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"icon_share" ofType:@"png"]];  
    }  
    else  
    {  
        webpage.thumbnailData = [NSData dataWithContentsOfURL:[NSURL URLWithString:picUrl]];  
    }  
    webpage.webpageUrl = url;  
    message.mediaObject = webpage;  
      
    WBSendMessageToWeiboRequest *request = [WBSendMessageToWeiboRequest requestWithMessage:message authInfo:authRequest access_token:myDelegate.wbtoken];  
    request.userInfo = @{@"ShareMessageFrom": @"SendMessageToWeiboViewController",  
                         @"Other_Info_1": [NSNumber numberWithInt:123],  
                         @"Other_Info_2": @[@"obj1", @"obj2"],  
                         @"Other_Info_3": @{@"key1": @"obj1", @"key2": @"obj2"}};  
      
    [WeiboSDK sendRequest:request];  
}  
  
@end
```

调用示例： 

```swift
- (void)shareViewButtonClick:(CustomShareType)shareType  
{  
    NSLog(@"%s",__func__);  
    NSLog(@"%lu",(unsigned long)shareType);  
      
    switch (shareType)  
    {  
        case ShareType_Weixin_Friend:  
        {  
            [JFShareTool wechatShareLinkWithScene:WXSceneSession url:url title:_shareTitle andDescription:_shareContent];  
            break;  
        }  
        case ShareType_Weixin_Circle:  
        {  
            [JFShareTool wechatShareLinkWithScene:WXSceneTimeline url:url title:_shareTitle andDescription:_shareContent];  
            break;  
        }  
        case ShareType_Weibo:  
        {  
            [JFShareTool weiboShareLinkWithUrl:url title:_shareTitle description:_shareContent andPicUrl:nil];  
            break;  
        }  
        case ShareType_QQ_Zone:  
        {  
            [JFShareTool qqShareLinkWithScene:QQShare_Qzone url:url title:_shareTitle andDescription:_shareContent];  
            break;  
        }  
        case ShareType_QQ_Friend:  
        {  
            [JFShareTool qqShareLinkWithScene:QQShare_Friends url:url title:_shareTitle andDescription:_shareContent];  
            break;  
        }  
        default:  
            break;  
    }  
}
```