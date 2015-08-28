title: 微信OAuth总结
date: 2015-08-14 17:50:38
categories: 微信开发
---
当用户从微信中打开网页时，基于微信提供的OAuth机制，可以获取到当前用户的基本信息，如昵称，头像等。这一能力是现在很多基于微信传播的网页的基础。本文总结这方面的一些心得技巧
<!--more-->

# 基本流程

首先需要登录微信公众号管理后台，配置允许跳转的域名。该域名必须是2级域名，不支持1级域名。所以数量有限，需要规划好。比如配置了a.exmaple.com为跳转域名，就无法再跳转到b.example.com了。如果这里配置错误的话，用户在跳转时，会得到一个语焉不详的错误提示“redirect_uri参数错误”，如果看到这个提示的话，多半就是OAuth配置的问题

配置之后，就可以使用微信的OAuth机制了，域名以a.example.com为例，假设实际的地址是a.example.com/abc.html，那么要配置成：
```
https://open.weixin.qq.com/connect/oauth2/authorize?appid=xxx&redirect_uri=http%3a%2f%2fa.example.com%2fabc.html&response_type=code&scope=snsapi_userinfo&state=STATE#wechat_redirect
```
然后用户在微信中打开访问此地址，微信就会提示用户鉴权，如果用户同意的话，则会跳转到redirect_uri指定的实际地址：
```
http://a.example.com/abc.html?code=CODE&state=STATE
```
然后：
1. 将code发到自己的服务器上
2. 通过code可以换到access_token和open_id
3. 用access_token和open_id，可以查到用户的基本信息

这里有几点容易产生误解：

1. 试图从前台页面通过ajax来换取access_token和open_id不可行，跨域问题
2. 这里虽然也叫access_token，但是与后台通过app_id和app_secret换到的access_token完全不是一个东西，不要搞混
3. 后台的API，也有获取用户基本信息的接口，但那个需要用户关注了公众号才能调用；而这个OAuth接口，只要用户同意授权，即使没有关注也可以调用，所以更加灵活

最后调用成功后，可以得到以下响应：
```
{
    "openid": "OPENID",
    "nickname": NICKNAME,
    "sex": "1",
    "province": "PROVINCE"
    "city": "CITY",
    "country": "COUNTRY",
    "headimgurl": "http://wx.qlogo.cn/mmopen/g3MonUZtNHkdmzicIlibx6iaFqAc56vxLSUfpb6n5WKSYVY0ChQKkiaJSgQ1dZuTOgvLLrhJbERQQ4eMsv84eavHiaiceqxibJxCfHe/46", 
	"privilege":[
	    "PRIVILEGE1"
	    "PRIVILEGE2"
    ],
    "unionid": "o6_bmasdasdsad6_2sgVt7hMZOPfL"
}
```

# 订阅号如何使用OAuth服务

当前微信限制了只有服务号才能使用OAuth。如果是订阅号的话，可以“借用”服务号来部分满足需求。虽然服务号得到的open_id并不是订阅号的open_id，但是可以用来区分是否是“同一个”用户。比如一个网页需要计数，可以通过这种方式标识出当前访问的用户

另外，如果订阅号和服务号已经在微信开放平台绑定到同一账号，那么union_id是一样的。不过现在大部分接口还是需要以open_id为参数，所以如果只知道union_id而不知道open_id，并没有太大用

# union_id机制

如果一个产品同时有订阅号、服务号、APP，又需要打通用户数据，识别出是同一个人，就需要用到union_id机制。在这种情况下，同一个用户相对订阅号、服务号、APP分别会有一个open_id，但是union_id是一样的，于是可以利用union_id建立关联关系。

```
id
union_id
dingyue_open_id
service_open_id
app_open_id
user_id
```
比如用上面这张表，无论用哪一个open_id，都能找到对应的其他open_id，以及业务系统中的user_id

# 静默授权

如果OAuth url串中，设置scope=snsapi_userinfo，则微信会提示用户授权，如下图
![wx oauth](http://kypic.oss-cn-hangzhou.aliyuncs.com/wxoauth.jpg)

只有用户同意授权，跳转回来的链接才会携带code参数，后续才能获得用户的基本信息。某些特殊场景，可以使用“静默授权”，用户的体验是立刻跳转到业务页面，不会看到授权页面：
1. 设置scope=snsapi_base，用户无感知，但是仅能获取到用户的open_id，无法获取其他信息
2. 对于已关注公众号的用户，如果用户从公众号的会话或者自定义菜单进入本公众号的网页授权页，即使是scope为snsapi_userinfo，也是静默授权，用户无感知