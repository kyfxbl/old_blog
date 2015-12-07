title: iOS输出星号
date: 2013-12-28 15:07
categories: iOS
---
如何在iOS中输出一个标准的星号
<!--more-->

想把UI调整成这个效果：

![](http://img.blog.csdn.net/20131228150518421?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

不知道怎么打出前面的星号，如果直接用shift + 8，效果很差

在stackoverflow找到了办法：[display a unicode character](http://stackoverflow.com/questions/7827962/how-to-locate-and-display-a-unicode-character-in-ios)

代码是：

```
prompt1.text = @"\u00B7 此手机号将作为您的用户名";
```