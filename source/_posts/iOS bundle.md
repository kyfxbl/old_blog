title: iOS bundle
date: 2013-12-02 15:57
categories: iOS 
---
iOS bundle就是app自身的文件夹
<!--more-->

想将app中的一个文件拷贝到documents目录，在网上找到了一段代码：

```
NSBundle *bundle=[NSBundle mainBundle];
NSString *zipFilePath=[bundle pathForResource:@"images" ofType:@"zip"];// Bundle内的images.zip文件
```

这里涉及到bundle的概念，不知道bundle是什么。又搜索 + debug了一番，原来bundle就是，应用本身那个app文件，但是其实是一个文件夹

![](http://img.blog.csdn.net/20131202155332453?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

红框框里就是所谓的bundle，每次编译以后生成的，至于要把哪些文件加到bundle里，则是在target里控制的：

![](http://img.blog.csdn.net/20131202155604453?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

pathForResource方法就是在动态确定某个资源文件的path，而不需要写死在代码里