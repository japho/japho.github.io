---
layout:     post
title:      iOS 微信第三方登录的实现 
subtitle:   原生微信SDK开发
date:       2016-04-18
author:     Japho
header-img: postimage/post_img_wechat.jpg
catalog: true
tags:
    - iOS
---

### 下载微信SDK

微信开放平台 [https://open.weixin.qq.com](https://open.weixin.qq.com)

![微信开放平台](http://upload-images.jianshu.io/upload_images/1269906-03c3fafce683fa65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 导入SDK

导入SDK，并添加依赖库

![](http://upload-images.jianshu.io/upload_images/1269906-4e5a0e9baf265421.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 配置URL scheme

在Xcode中，选择你的工程设置项，选中`TARGETS`一栏，在`info`标签栏的`URL type`添加`URL scheme`为你所注册的应用程序id（如下图所示），此步为配置应用间的跳转。

![配置Url Scheme](http://upload-images.jianshu.io/upload_images/1269906-c20b3c9b67a92432.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 开始编写代码

- 在`Appdelegate.h`中引用微信文件，声明遵循代理。

```swift

#import <UIKit/UIKit.h>  
#import "WXApi.h"  
  
@interface AppDelegate : UIResponder <UIApplicationDelegate,WXApiDelegate>  
  
@property (strong, nonatomic) UIWindow *window;  
  
@end


```

- `注册SDK`要使你的程序启动后微信终端能响应你的程序，必须在代码中向微信终端注册你的id。（在 `AppDelegate` 的 `didFinishLaunchingWithOptions` 函数中向微信注册`id`）。


```swift

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {  
    // Override point for customization after application launch.  
      
    [WXApi registerApp:@"这里填写你的AppID"];  
      
    return YES;  
}  


```

- 重写`AppDelegate`的`handleOpenURL`和`openURL`方法


```swift

//和QQ,新浪并列回调句柄  
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation  
{  
    return [WXApi handleOpenURL:url delegate:self];  
}  
  
- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url  
{  
    return [WXApi handleOpenURL:url delegate:self];  
}  


```

- 调用微信API，调用微信登录

>`scope` : 应用授权作用域，如获取用户个人信息则填写`snsapi_userinfo`
>
>`state` : 用于保持请求和回调的状态，授权请求后原样带回给第三方。该参数可用于防止csrf攻击（跨站请求伪造攻击），建议第三方带上该参数，可设置为简单的随机数加session进行校验（非必填）。

```swift

- (void)wechatLoginButtonPressed  
{  
    NSLog(@"%s",__func__);  
      
    //构造SendAuthReq结构体  
    SendAuthReq* req =[[SendAuthReq alloc] init];  
    req.scope = @"snsapi_userinfo" ;  
    req.state = @"123" ;  
    //第三方向微信终端发送一个SendAuthReq消息结构  
    [WXApi sendReq:req];  
}  

```

- 在`Appdelegate`中实现微信的代理，获取微信返回的`code`，这里我使用了通知的方法来调用登录`controller`中的相应才做，也可以使用代理、`KVO`等方式来实现。

```swift

//授权后回调 WXApiDelegate  
-(void)onResp:(BaseReq *)resp  
{  
    /* 
     ErrCode ERR_OK = 0(用户同意) 
     ERR_AUTH_DENIED = -4（用户拒绝授权） 
     ERR_USER_CANCEL = -2（用户取消） 
     code    用户换取access_token的code，仅在ErrCode为0时有效 
     state   第三方程序发送时用来标识其请求的唯一性的标志，由第三方程序调用sendReq时传入，由微信终端回传，state字符串长度不能超过1K 
     lang    微信客户端当前语言 
     country 微信用户当前国家信息 
     */  
  
    if ([resp isKindOfClass:[SendAuthResp class]]) //判断是否为授权请求，否则与微信支付等功能发生冲突  
    {  
        SendAuthResp *aresp = (SendAuthResp *)resp;  
        if (aresp.errCode== 0)  
        {  
            NSLog(@"code %@",aresp.code);  
            [[NSNotificationCenter defaultCenter] postNotificationName:@"wechatDidLoginNotification" object:self userInfo:@{@"code":aresp.code}];  
        }  
    }  
}  


```

相对应注册通知方法：

```swift

 //跳转到主界面
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(wechatDidLoginNotification:) name:@"wechatDidLoginNotification" object:nil];


```

记得在dealloc方法中移除通知，避免发生冲突：


```swift

- (void)dealloc  
{  
    [[NSNotificationCenter defaultCenter] removeObserver:self name:@"wechatDidLoginNotification" object:nil];  
}  


```

**微信登录认证后获取用户的个人信息比较麻烦，需要三个步骤：**

>- 获取微信登录code
- 根据code获取accessToken和openId
- 根据accessToken和openId获取用户信息

**具体步骤：**

刚刚我们已经在`appdelegate`中微信的代理中获取到了`code`，下面直接来进行第二步，根据`code`获取`accessToken`和`openId`：
参照微信开放平台官方文档：

![](http://upload-images.jianshu.io/upload_images/1269906-afc238a8d21aac3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```swift

- (void)getWechatAccessTokenWithCode:(NSString *)code  
{  
    NSString *url =[NSString stringWithFormat:  
      @"https://api.weixin.qq.com/sns/oauth2/access_token?appid=%@&secret=%@&code=%@&grant_type=authorization_code",  
        WechatAppKey,WechatSecrectKey,code];  
      
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{  
        NSURL *zoneUrl = [NSURL URLWithString:url];  
        NSString *zoneStr = [NSString stringWithContentsOfURL:zoneUrl encoding:NSUTF8StringEncoding error:nil];  
        NSData *data = [zoneStr dataUsingEncoding:NSUTF8StringEncoding];  
        dispatch_async(dispatch_get_main_queue(), ^{  
              
            if (data)  
            {  
                NSDictionary *dic = [NSJSONSerialization JSONObjectWithData:data   
                                                                    options:NSJSONReadingMutableContainers error:nil];  
                  
                NSLog(@"%@",dic);  
                NSString *accessToken = dic[@"access_token"];  
                NSString *openId = dic[@"openid"];  
                  
                [self getWechatUserInfoWithAccessToken:accessToken openId:openId];  
            }  
        });  
    });  
}  


```

现在已经获取了微信的`accessToken`和`openId`，就可以请求相应的微信接口了。
参照文档，我们使用以下接口：

![](http://upload-images.jianshu.io/upload_images/1269906-4350c96c94ce8487.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用这个接口就可以获取用户信息了，然后调用自己的方法进行登录，这里可以使用`openId`当做账号，他是每个微信用户唯一对应的`id`

```swift

- (void)getWechatUserInfoWithAccessToken:(NSString *)accessToken openId:(NSString *)openId  
{  
    NSString *url =[NSString stringWithFormat:  
                    @"https://api.weixin.qq.com/sns/userinfo?access_token=%@&openid=%@",accessToken,openId];  
      
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{  
        NSURL *zoneUrl = [NSURL URLWithString:url];  
        NSString *zoneStr = [NSString stringWithContentsOfURL:zoneUrl encoding:NSUTF8StringEncoding error:nil];  
        NSData *data = [zoneStr dataUsingEncoding:NSUTF8StringEncoding];  
        dispatch_async(dispatch_get_main_queue(), ^{  
              
            if (data)  
            {  
                NSDictionary *dic = [NSJSONSerialization JSONObjectWithData:data   
                                                                    options:NSJSONReadingMutableContainers error:nil];  
                  
                NSLog(@"%@",dic);  
                  
                NSString *openId = [dic objectForKey:@"openid"];  
                NSString *memNickName = [dic objectForKey:@"nickname"];  
                NSString *memSex = [dic objectForKey:@"sex"];  
                  
                [self loginWithOpenId:openId memNickName:memNickName memSex:memSex];  
            }  
        });  
          
    });  
} 


```

至此，就实现了简单的微信登录

效果图：

![](http://upload-images.jianshu.io/upload_images/1269906-7adef59824ff8291.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
