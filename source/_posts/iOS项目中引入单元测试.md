title: iOS项目中引入单元测试
date: 2013-12-24 17:53
categories: iOS 
---
我们的项目在没有单元测试的情况下“裸奔”了一个月，今天决定将单元测试加进来
<!--more-->

先在网上搜索了一下，发现有3个unit test的框架：XCTEST，OCTEST，GHTEST。由于也不是很了解，就先用XCTEST凑合一下，毕竟是xcode自带的，应该集成会比较容易点

首先在工程里add target

![](http://img.blog.csdn.net/20131224173846921?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

然后选择Cocoa touch unit test bundle

![](http://img.blog.csdn.net/20131224174011750?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这里type我选择的是XCTest

![](http://img.blog.csdn.net/20131224174218156?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

然后会生成一个新的target，以及自动创建单元测试文件夹

![](http://img.blog.csdn.net/20131224174409343?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

由于我们这个项目是一个Cordova工程，所以编译会报错，接下来要在这个新的target里，把所需的framework加上。我这里增加了以下4个：

CoreLocation.framework

AssetsLibrary.framework

CoreGraphics.framework

MobileCoreServices.framework

编译就可以通过了，根据不同的项目，可能需要添加别的库

![](http://img.blog.csdn.net/20131224174757375?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

然后在左侧的Test Navigator里，就可以运行单元测试了。当然还需要增加新的单元测试，用command + N，然后增加测试类就可以了

我不太清楚ios里的最佳实践是什么样的，所以还是延续做java开发时的习惯，在Tests目录下，创建跟src下同名的文件夹，测试类的命名，在原始类名的基础上，增加Test后缀

# 单元测试类依赖原始类

这是最普遍的场景，如果做不到，根本就谈不上单元测试了。一般单元测试的代码都会这么写：

```
@interface YLSClientInfoTest : XCTestCase

@end

@implementation YLSClientInfoTest

{
    YLSClientInfo *clientInfo;
}

- (void)setUp
{
    [super setUp];
    clientInfo = [YLSClientInfo new];
}
```
单元测试类里有一个原始类的引用，并在setUp()方法中创建实例和初始化，然后在具体的测试方法testXXX()中做测试

本来我以为这个不需要特别的配置，谁知一运行就报错：

```
Undefined symbols for architecture i386: "_OBJC_CLASS_$_YLSClientInfo", referenced from: objc-class-ref in YLSClientInfoTest.o (maybe you meant: _OBJC_CLASS_$_YLSClientInfoTest)
```

看起来是找不到YLSClientInfo这个类。搞了好一会，原来这个类没有被编译到Test的Target里，需要在Util Pane里打钩才行：

![](http://img.blog.csdn.net/20131224201714406?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

所以需要做单元测试的类，以及相关依赖的类，都要打上勾才行。一个个打钩很麻烦，还容易遗漏，可以在Build Phases的Compile Sources里，一次性都加上

# 解决对Pods的依赖

勾上以后，一跑又挂了，这次换了个错误信息，说找不到FMDatabase.h，这个是项目通过CocoaPods引入的

这次自己研究了半天也没找到地方，最后还是在stackoverflow里找到答案

在configurations里，把base从none改为Pods

![](http://img.blog.csdn.net/20131224203033000?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

所有通过CocoaPods引入的第三方组件，只要引入了pods编译后的库就行了。直接导入工程的组件，则都需要手工加到Test target里才可以。所以用pods还是要方便多了