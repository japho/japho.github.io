---
layout:     post
title:      iOS QQ第三方登录的实现
subtitle:   基于QQ原生SDK开发实现
date:       2016-04-14
author:     Japho
header-img: https://ws4.sinaimg.cn/large/006tNbRwly1fxi1vmy4boj31gs0o6dgj.jpg
catalog:    true
tags:
    - iOS
---

### 前言

>平时我们经常会在一些app的登录界面中见到第三方登录，一些应用中一般会使用一些类似`shareSDK`的集成平台，他们是将QQ、微信、微博等第三方进行了二次封装，灵活性不太高，其实直接集成也是比较容易的。今天就来简单的说一下QQ的第三方登录的集成。

### 注册开发者账号

腾讯开放平台 [http://open.qq.com/](http://open.qq.com/)

### 下载所需SDK

SDK下载地址：[http://wiki.open.qq.com/wiki/mobile/SDK](http://wiki.open.qq.com/wiki/mobile/SDK)下载

![下载SDK](http://upload-images.jianshu.io/upload_images/1269906-09d6d16aa6ade24b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### SDK的目录结构

![目录结构](http://upload-images.jianshu.io/upload_images/1269906-c55d70eac31d40eb.png)

>sample：示例代码
>
>1. `TencentOpenAPI.framework`打包了iOS SDK的头文件定义和具体实现。
>2. `TencentOpenApi_iOS_Bundle.bundle` 打包了iOS SDK需要的资源文件。

### 创建项目导入SDK添加依赖库

- 将`TencentOpenAPI.framework`和`TencentOpenApi_iOS_Bundle.bundle`拖入工程，注意勾选`copy items if needed`

![添加依赖库](http://upload-images.jianshu.io/upload_images/1269906-51289c4fbecd86b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 点击`Project navigator` 点击`TARGETS` --->  `General`  ---> `Linked Frameworks and Libraries`

- 点击加号添加

![添加依赖库](http://upload-images.jianshu.io/upload_images/1269906-2b3492004d2d24be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 添加SDK依赖的系统库文件。分别是

>`”Security.framework”`,
>`“libiconv.dylib”`,
>`“SystemConfiguration.framework”`,
>`“CoreGraphics.Framework”`,
>`“libsqlite3.dylib”`,
>`“CoreTelephony.framework”`,
>`“libstdc++.dylib”`,
>`“libz.dylib”`

### 修改工程配置属性

- 在工程配置中的`Build Settings`一栏中找到`Linking`配置区，给`Other Linker Flags`配置项添加属性值`-fobjc-arc`

![修改工程配置](http://upload-images.jianshu.io/upload_images/1269906-b7c82914bc009b68.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

配置之后：

![修改工程配置](http://upload-images.jianshu.io/upload_images/1269906-f570fc54824ffcfe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 在XCode中，选择你的工程设置项，选中`TARGETS`一栏，在“info”标签栏的`URL type`添加一条新的`URL scheme`，新的`scheme = tencent + appid`（例如你的`appid`是`123456` 则填入`tencent123456`） `identifier` 填写：`tencentopenapi`。

`注意：`此处的配置是实现应用间的跳转即跳转至QQ进行授权以及跳转回app进行其他的操作。

![修改工程配置](http://upload-images.jianshu.io/upload_images/1269906-cb563972e0817714.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 创建应用

在腾讯开发者平台中创建应用，获得`appId`及`appKey`，具体步骤详见开发中心，这里不再赘述。

### 开始添加代码

做好了之前的步骤，现在开始添加调用授权的代码。

`AppDelegate.m:`


```swift

#import "AppDelegate.h"  
#import <TencentOpenAPI/TencentOAuth.h> //导入头文件  
  
@interface AppDelegate () <TencentSessionDelegate>  
{  
    TencentOAuth *_tencentOAth;//创建授权对象  
}  
@end  
  
@implementation AppDelegate  
  
  
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {  
    // Override point for customization after application launch.  
      
    _tencentOAth = [[TencentOAuth alloc] initWithAppId:@"这里填写你申请的appID" andDelegate:self];  
      
    return YES;  
}  
  
//重写appDelegate中的回调方法  
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation {  
      
    return [TencentOAuth HandleOpenURL:url];  
}  
  
- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url {  
      
    return [TencentOAuth HandleOpenURL:url];  
}  


```

你所要调用第三方登录的controller

```swift

#import "ViewController.h"  
#import <TencentOpenAPI/TencentOAuth.h>  
  
@interface ViewController () <TencentSessionDelegate>  
{  
    TencentOAuth *_tencentOAth;  
}  
@end  
  
@implementation ViewController  
  
- (void)viewDidLoad {  
    [super viewDidLoad];  
    // Do any additional setup after loading the view, typically from a nib.  
}  
  
- (void)didReceiveMemoryWarning {  
    [super didReceiveMemoryWarning];  
    // Dispose of any resources that can be recreated.  
}  
  
- (IBAction)qqLogin:(id)sender  
{  
    _tencentOAth = [[TencentOAuth alloc] initWithAppId:@"你所申请的appID" andDelegate:self];  
      
    //设置需要获取的权限列表，需要什么就写什么  
    NSArray *permissions = [NSArray arrayWithObjects:@"get_user_info",@"get_simple_userinfo",@"add_t",nil];  
  
    //调用此方法开始跳转进行授权  
    [_tencentOAth authorize:permissions inSafari:NO];  
}  
  
#pragma mark -- TencentSessionDelegate  
  
//登陆完成调用  
  
- (void)tencentDidLogin  
  
{  
    if (_tencentOAth.accessToken &&0 != [_tencentOAth.accessToken length])  
    {  
        //这里可获取accessToken,Access Token凭证，用于后续访问各开放接口  
        NSLog(@"accessToken %@",_tencentOAth.accessToken);  
  
        //这里可获取openID,openId是用户授权登录后对该用户的唯一标识  
        NSLog(@"openId %@",_tencentOAth.openId);  
          
        //获取用户信息  
        [_tencentOAth getUserInfo];  
    }  
    else  
    {  
        NSLog(@"登录不成功没有获取accesstoken");  
    }  
      
}  
  
  
//非网络错误导致登录失败：  
-(void)tencentDidNotLogin:(BOOL)cancelled  
  
{  
      
    NSLog(@"tencentDidNotLogin");  
      
    if (cancelled)  
    {  
        NSLog(@"用户取消登录");  
    }  
    else  
    {  
        NSLog(@"登录失败");  
    }  
      
}  
  
// 网络错误导致登录失败：  
- (void)tencentDidNotNetWork  
  
{  
    NSLog(@"tencentDidNotNetWork");  
}  
  
- (void)getUserInfoResponse:(APIResponse *)response  
  
{  
    NSLog(@"respons:%@",response.jsonResponse);  
      
    NSData *imageData = [NSData dataWithContentsOfURL:[NSURL URLWithString:response.jsonResponse[@"figureurl_2"]]];  
    UIImage *image = [UIImage imageWithData:imageData scale:1];  
    self.headerImageView.image = image;  
      
    self.nameLabel.text = response.jsonResponse[@"nickname"];  
    self.sexLabel.text = response.jsonResponse[@"gender"];  
    self.provinceLabel.text = response.jsonResponse[@"province"];  
    self.cityLabel.text = response.jsonResponse[@"city"];  
}  
  
@end  


```

**具体步骤再来叙述一下：**

- 初始化TencentOAuth 对象 appid来自应用宝创建的应用， deletegate设置为self 记得要实现代理方法
- 设置需要获取的权限列表，需要什么就写什么
- 调用方法开始跳转进行授权 [_tencentOAth authorize:permissions inSafari:NO];
- 在tencentDidLogin代理中获取openId、acess_token
- 调用[_tencentOAth getUserInfo] 获取用户的信息
- 在 - (void)getUserInfoResponse:(APIResponse *)response 代理中获取用户的信息。

至此，QQ的授权登录就算是基本的实现了，至于具体的其他注册逻辑就需要自己进行改写了。
