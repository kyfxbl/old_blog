title: iOS7图片资源命名规则
date: 2014-01-19 23:50
categories: iOS 
---
今天需要将2张图片放到工程里，本以为是1分钟就能搞定的事情，最后居然弄了2个小时，把折腾的过程记录下来，以免下次再浪费时间
<!--more-->

其实失败的原因是，同事给的图片，后缀是.png，但是其实是一个jpg文件，所以xcode在build的时候报错了。开始我们是把图片往Images.xcassets里拷，所以xcode不报错，最后实在没办法，单独拷这个文件，xcode才报错了，提示not a PNG file，这才最后找到了原因。这一点上xcode比较不人性化

顺便了解了一下ios应用中图片的命名规则：

name.png 普通图片

name@2x.png 视网膜屏图片

name~ipad.png ipad图片

name@2x~ipad.png ipad视网膜屏图片

也就是说，如果图片的原始文件名是filename，则根据需要，准备@2x和~ipad后缀的文件就可以了。在应用运行时，ios会根据硬件的情况，自动去加载合适的图片文件。比如如果应用当前跑在ipad air retina上，则系统会去加载name@2x~ipad.png；如果是跑在iphone4上，则会自动加载name.png

这个只是文件的命名，并不保证分辨率，图片应该具备多少分辨率，跟命名无关

最后，在程序中加载图片，下面2行代码都可以：

```
[UIImage imageNamed:@"filename"];// 根据硬件，自动加载filename@2x~ipad.png
[UIImage imageWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"filename" ofType:@"png"]];
```
imageNamed:方法明显更方便点，并且此方法是带缓存的