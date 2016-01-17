title: iOS开发的兼容和适配问题
date: 2014-10-20 23:47
categories: iOS
---
由于苹果公司的霸道作风，每个iOS开发者都会面对不同设备和版本的兼容问题。基本上升级一次xcode鸡飞狗跳是非常正常的，稍微总结一下需要注意的几个方面
<!--more-->

# 硬件ARCH

这个直接决定了APP能不能跑在指定设备上。现在常见的ARCH有3种：

armv7，对应iPhone4，iPhone4S，iPad mini，iPad3

armv7s，对应iPhone5，iPhone5C，iPad4

arm64，对应iPhone5S，iPad Air，iPad mini2，iPhone6，iPhone6 Plus

ARCH是向下兼容的，armv7生成的二进制包，可以跑在arm64的设备上，只是有些优化不可用；反之，arm64生成的二进制包，就不能跑在iPhone4和iPhone5上了

同时，build配置项中有一个Build Active Architecture Only的选项，在release中设置成NO，则会编译出混合二进制包。比如Architectures指定了armv7和arm64，则打出的是一个混合二进制包，既可以跑在armv7设备上，也可以优化跑在arm64设备上

鉴于现在armv7的设备还是占有相当的比例，所以build的时候，一定还是需要包含armv7选项的

另外一个问题，就是用armv7和armv7s打出的Library，是无法跟arm64打出的二进制包link的。所以如果是开发SDK供其他开发者使用，那么build的时候也一定要包含arm64。否则如果SDK使用者的build中包含arm64，Library就无法使用了。所以做SDK开发，需要把所有ARCH都选上

# iOS SDK版本

另外就是iOS版本的问题，双向都需要考虑兼容

用iOS7 SDK编译的APP，在iOS8平台上，很有可能不能正常运行。比如在几个UIView中共享UIDatePicker，这种行为在iOS7下是OK的，但是在iOS8下就会crash；比如iOS7下的auto sizing，在iOS8下也可能出现显示错误。因此，iOS8操作系统出来，旧的APP基本上需要重新适配，再次发布

反之，iOS8 SDK编译的APP，在iOS7系统上，也经常不能正常运行。比如我使用了UIAlertController类，在iOS7下就会直接飞掉，报错：

```
**Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: 'Application tried to present a nil modal view controller on target**
```

也有一些iOS8下的API，在iOS7下不会crash，但是毫无反应。所以，经常需要一些版本兼容的代码，例如：

```
if ([[[UIDevice currentDevice] systemVersion] floatValue] >= 8.0) {
    // for iOS8
}else{
    // for iOS7
}
```

# 屏幕尺寸

即使ARCH和API的适配问题都解决，UI的适配还是一个更大的挑战，一方面是各种设备的尺寸都不同，比如同一个页面，需要考虑在480，568，1024的高度下都正常显示；另一方面，不同的iOS版本，在细节方面也会有细微的差别，比如toolbar和navigation bar的处理等等