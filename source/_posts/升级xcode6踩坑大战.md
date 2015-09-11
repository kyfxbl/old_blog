title: 升级xcode6踩坑大战
date: 2014-10-15 23:07
categories: iOS
---
昨天升级到xcode6，踩了几个坑，主要是link的时候各种Undefined symbols for architecture。把遇到的问题总结一下
<!--more-->

# Cocoapods的问题

先是pods编译出来的libPods.a失效了，仔细看了一下build日志，有这样一行不显眼的提示：

Pods was rejected as an implicit dependency for ‘libPods.a’ because its architectures ‘armv7s’ didn’t contain all required architectures ‘arm64’

原因是xcode6工程默认的Architectures配置了arm64，而老版本的pods只配置了armv7和armv7s，因此link error

于是我升级了pods，最后又有这样一行不显眼的提示：

```
The `NailShop [Debug]` target overrides the `OTHER_LDFLAGS` build setting defined in `Pods/Target Support Files/Pods/Pods.debug.xcconfig'. This can lead to problems with the CocoaPods installation

    - Use the `$(inherited)` flag, or

    - Remove the build settings from the target.
```

按它的提示操作，其实也有坑。首先在xcode6里，我没有找到OTHER_LDFLAGS配置项，似乎是被Other Linker Flags替换了；然后，按它说的删除这个配置项也是不行的，而是要设置成$(inherited)，正确配置的话，该项会自动填上值：

![](http://img.blog.csdn.net/20141015225636171)

如果直接删掉的话，这里是空白，也是不行的

升级以后，pods的Architectures会配置成armv7和arm64（Valid Architectures是armv7 armv7s arm64，不要紧），Build Active Architecture Only的debug是YES，release是NO。而主项目也应该配置成同样的值

至此，libPods.a可以被正常link和加载了

# 依赖Framework的问题

有一个UIView使用了OpenGL，在xcode5不需要额外import，但是在xcode6里，编译不通过，需要import头文件：

```
#import <OpenGLES/ES2/glext.h>
```
然后又是Undefined symbols for architecture，原来是少引入了一个framework。增加这行import之后，需要在build时加入OpenGLES.framework

# cordova的问题

终于build成功以后，刚进APP就crash了，打上exception breakpoint，发现是cordova的代码不兼容，升级cordova到3.6.0以后，问题解决

# 各种deprecated问题

茫茫多的黄色感叹号，这个问题就是发现一处改一处，替换成iOS8 SDK要求的写法