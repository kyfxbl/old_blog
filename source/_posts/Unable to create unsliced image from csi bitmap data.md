title: Unable to create unsliced image from csi bitmap data
date: 2015-06-04 21:08
categories: iOS
---
xcode有这个BUG，截止到6.3.1还没有修复
<!--more-->

当打ipa包，需要支持iOS7的设备时，xcode不会把Images.xcassets里的.jpg图片正确打包

现象是在iOS8上可以正确显示的图片，在iOS7上会显示白屏，并且console报错：Unable to create unsliced image from csi bitmap data，解决办法是使用.png图片，或者把.jpg图片的扩展名改成.png