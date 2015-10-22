title: 从web页面打开iOS应用
date: 2015-10-21 17:56:58
categories: iOS
---
本文介绍从web页面打开iOS app的方法
<!--more-->

从web页面打开app是一个常见场景，大致上有2种做法

# 利用Safari原生Banner

只需要在html中加入一段meta，即可在Safari中显示一个Banner。如果未安装此app，会跳转到app store的下载页面，否则会直接打开应用

效果图：

![Safari Banner](https://developer.apple.com/library/ios/documentation/AppleApplications/Reference/SafariWebContent/Art/smartbanner_2x.png)

html代码如下：
```
<meta name='apple-itunes-app' content='app-id=你的应用的app-id'>
```

另外我在简书的网站上，看到代码是这样写的：
```
<meta name="apple-itunes-app" content="app-id=888237539, app-argument=jianshu://notes/2283513">
```

特别的地方在于，多了一个app-argument参数，可能可以传递到app的这个方法里进行处理：
```
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString*, id> *)options
```

# 使用自定义链接

第一种方法的好处是方便，但是缺点是样式是固定的，不能自定义，所以更好的办法是采用自定义链接。代码如下：

```
<a href="https://itunes.apple.com/cn/app/id995195037" id="openApp">打开应用</a>

<script type="text/javascript">

    document.getElementById('openApp').onclick = function(e){

    // 通过iframe的方式尝试打开APP，如果能正常打开，会直接切换到APP，并自动阻止a标签的默认行为
    // 否则打开a标签的href链接
    var ifr = document.createElement('iframe');
    ifr.src = 'com.yilos.nailstar://topic/abcdefg';
    ifr.style.display = 'none';
    document.body.appendChild(ifr);

    window.setTimeout(function(){
        document.body.removeChild(ifr);
    },3000)
};
</script>
```

本质上是发起了一个请求：
```
com.yilos.nailstar://topic/abcdefg
```

这个请求会在iOS系统中查找对应的url schema，然后打开此应用，同样会进入以下方法：
```
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString*, id> *)options
```
所以有机会对自定义的参数进行处理

比较巧妙的地方在于，打开app的行为，会阻止a链接的默认跳转行为；而如果打开失败，则会进入app store的下载页面

简书的代码是利用同样的原理：
```
if (/iphone|ipad|ipod/.test(ua)) {

    // in iOS
    if (ua.match(/MicroMessenger/i) || ua.match(/weibo/i)){
    
        Maleskine.showWeixinHelp();
        
    }else if (ua.match(/MQQBrowser/i) || ua.match(/QQ/i)) {
            
        Maleskine.showQQHelp();
          
    } else {
            
        window.location = "jianshu://p/12345678";
        window.setTimeout(function() {
            window.location = "https://itunes.apple.com/cn/app/jian-shu/id888237539?ls=1&amp;mt=8";
        }, 400);
    }
}
```
也是先尝试打开app，如果打开失败，就跳转到app store下载页面

# 微信的兼容性问题

微信做了特殊处理，在微信中打开的web页，既不能跳转到app，也不能跳转到app store

所以一般的做法是提示用户在Safari中打开

