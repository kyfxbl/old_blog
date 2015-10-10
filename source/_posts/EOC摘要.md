title: EOC摘要
date: 2015-04-20 00:42
categories: iOS
---
今天有事回老家，在动车上把EOC囫囵吞枣看完了，晚上回来摘要几条觉得比较重要的
<!--more-->

1、在可能的时候，尽量多用字面量

2、尽可能为对象实现description方法

3、私有方法的前缀，不使用单个下划线，会跟苹果冲突

4、如果一个类里的方法太多，考虑用category分类

5、category也要加前缀，因为oc缺少完备的namespace方案，所以在可能冲突的地方，都要考虑加前缀

6、以弱引用避免retain cycle

7、ARC无法处理CoreFoundation框架的对象回收，需要自己调用CFRelease方法

8、定位内存问题时，可以用xcode的zombie object功能

9、用typedef为block命名，使之易读可复用

10、performSelector系列方法可以当做不存在了

11、GCD不是万能的，有时候还需要NSOperation