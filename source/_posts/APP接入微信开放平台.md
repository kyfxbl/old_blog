title: APP接入微信开放平台
date: 2014-08-06 12:21
categories: iOS
---
简介APP接入微信开放平台
<!--more-->

# 开放平台和公众平台的区别

公众平台针对的是公众账号，除了提供管理后台之外，也开放了若干接口，让微信服务器和开发者自己的应用系统能够对接

开放平台是微信的整体接入方式，不局限于公众账号（订阅号，服务号），移动APP和web应用也可以通过开放平台，实现与微信对接。可以说，公众平台是开放平台的一个子集

开放平台大体上分为3个部分，分别针对移动APP，WEB应用，公众账号的接入

# 移动APP接入开放平台的作用

目前，移动APP接入微信开放平台后，可以获得以下的特性：

1、向微信好友发消息

2、发消息到朋友圈

3、收藏内容到“我的收藏”

4、用微信账号登陆APP，获得微信账号的信息

5、支持微信支付

在朋友圈可以看到一个消息后面跟着“来自XXX”，这就是XXX应用接入开放平台后得到的能力

# ios app接入方式

流程和代码都不复杂，具体方法请看开放平台官网，本文不赘述。只提醒一点，需要在xcode里配置你自己APP的URL Type，URL Schemas需要填写微信开放平台提供的那个app id

如果漏掉了这一步，一样可以发消息到微信，但是发完消息以后就无法从微信再跳转回你的APP了，因为微信客户端也是通过openURL方法，跳回你的APP，需要你的APP自己注册上URL Schemas

# 对接微信的原理

首先，一个大的限制是，APP不可能通过微信提供的SDK，直接把消息发到微信服务器上。而是从开发者的APP中，打开微信应用，然后还是由微信把消息发出去，再跳回开发者自己的APP

也就是说，APP和微信的交互，是通过应用间跳转来完成的，所以核心还是iOS的这2个方法：

```
- (BOOL)openURL:(NSURL*)url;
```

```
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
```

发送消息的代码是：

```
[WXApi sendReq:req];
```

微信SDK不开源的，但是很容易想到，跳转到另一个app的方式在iOS中就是openURL方法，所以这行代码做的事情，类似于：

```
NSString *weixinURL = @"weixin_schema://app_id?title=xxx&content=xxx";
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:weixinURL]];
```

通过weixin_schema，打开了机器上安装的微信应用；在URL末尾添加了相关参数，微信解析后处理。然后在微信里把消息发出去以后，微信也会调用openURL，又回到了开发者自己的APP。URL地址是：wx_xxxxxxxxxxx://platformId=wechat

这个URL被AppDelegate中的这个方法拦截：

```
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    return [WXApi handleOpenURL:url delegate:self];
}
```

然后handleOpenURL方法调用了：

```
-(void) onResp:(BaseResp*)resp
{
    NSString *strTitle = [NSString stringWithFormat:@"发送消息结果"];
    NSString *strMsg = [NSString stringWithFormat:@"errcode: %d", resp.errCode];

    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:strTitle message:strMsg delegate:nil cancelButtonTitle:@"OK" otherButtonTitles:nil];
    [alert show];
}
```

整个交互的原理就是这样。具体到对接的代码里，主要是2个流程：

1、应用主动发消息给微信。应用调用sendReq方法，然后在onResp方法里处理微信的响应

2、微信发消息给应用。应用在onReq里处理微信的请求，然后调用sendResp方法发响应到微信

然后这2个流程里用到的参数，都是微信SDK里提供的封装类，如SendMessageToWXReq，WXMediaMessage等

# 对接微信的限制

如上所述，由于SDK并没有提供应用直接发送请求到微信服务器的能力，而只能带参数跳转到微信APP，所以接入的限制还是比较大的，很多事情都做不了

比如说，用户的设备上一定要装有微信，而且已经处于登陆状态。因此很多for iPad的APP，就很难对接微信。因为会在iPad上安装微信的用户是很少的，一般都是装在手机上

还有，也无法实现在自己的APP里选定用户发送，只能是编辑好内容，跳到微信里，在微信通讯录里选要发送的好友

也不能根据手机号，直接向微信账号发送申请加为好友的请求

尽管如此，对接微信之后，对APP的社交传播还是有较大的价值，所以现在可以看到大部分的APP，都有接入微信的功能