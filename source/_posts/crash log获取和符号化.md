title: crash log获取和符号化
date: 2014-05-22 17:01
categories: iOS
---
本文介绍如何获取iOS app的crash log，并符号化
<!--more-->

# 获取crash log

如果不借助第三方框架，要收集ios app的crash log是一件很困难的事情。有2个办法：

第一个办法是要求用户打开“诊断与用量”中的自动发送，然后如果APP崩溃了，ios会弹出提示框，用户确认之后，crash log会自动发送到苹果后台，然后用开发者账号登陆上去，可以拿到crash log

第二个办法是将device，同步到iTunes之后，再从pc上拿到crash log，再发送

详细步骤见：
[how to get crash log](https://forums.adobe.com/docs/DOC-4341)
[syncing with iTunes](http://support.apple.com/kb/ht1386)

可以看到，2种方式都不太靠谱，所以现实的方法还是使用第三方框架，比如Crashlytics之类的。在app崩溃的时候，会自动采集crash log

# 符号化

拿到crash log以后，还要解决怎么读的问题。原始的crash log是不可读的，内容类似这样：

Last Exception Backtrace: 
(0x313b1e83 0x3ba4d6c7 0x312e7d95 0xcef95 0xce843 0x7c6d1 0x3bf320c3 0x3bf377d9 0x3bf379c5 0x3c061dff 0x3c061cc4) 

Thread 0: 
0   libsystem_kernel.dylib         0x3bfeaa84 0x3bfea000 + 2692 
1   libsystem_kernel.dylib         0x3bfea87d 0x3bfea000 + 2173 
2   CoreFoundation                 0x3137c555 0x312dd000 + 652629 
3   CoreFoundation                 0x3137acbb 0x312dd000 + 646331 
4   CoreFoundation                 0x312e546d 0x312dd000 + 33901 
5   CoreFoundation                 0x312e524f 0x312dd000 + 33359 
6   GraphicsServices               0x35fe62e7 0x35fdf000 + 29415 
7   UIKit                          0x33b9a841 0x33b2b000 + 456769

这样的一份日志，即使拿到了也几乎没有意义，对定位问题没有帮助，所以需要将crash log做符号化

一般来说，任何app的crash log在xcode中都能部分符号化，也就是UIKit，CoreFoundation这种系统级的库，但是要想真正得到完全符号化的crash log，需要满足2个条件：

1、有app的二进制包

2、有编译此二进制包时得到的dSYM文件

只要是曾经编译过该app的机器，应该都满足这个条件。然后把拿到的.ips或.crash文件导入到xcode，就会自动符号化了，完全符号化以后，crash log就可读了：

Last Exception Backtrace: 
0   CoreFoundation                  0x183cba950 __exceptionPreprocess + 132 
1   libobjc.A.dylib                 0x1905701fc objc_exception_throw + 60 
2   CoreFoundation                  0x183bbc218 -[__NSArrayI objectAtIndex:] + 204 
3   TestNotification                0x100029cfc -[MainViewController clicked1] (MainViewController.m:42) 
4   UIKit                           0x186cb90c8 -[UIApplication sendAction:to:from:forEvent:] + 100 
5   UIKit                           0x186cb905c -[UIApplication sendAction:toTarget:fromSender:forEvent:] + 24 
6   UIKit                           0x186ca2538 -[UIControl _sendActionsForEvents:withEvent:] + 376 
7   UIKit                           0x186cb8a5c -[UIControl touchesEnded:withEvent:] + 584 
8   UIKit                           0x186cb86f0 -[UIWindow _sendTouchesForEvent:] + 692 
9   UIKit                           0x186cb3388 -[UIWindow sendEvent:] + 1172 
10  UIKit                           0x186c84b68 -[UIApplication sendEvent:] + 256 
11  UIKit                           0x186c82c58 _UIApplicationHandleEventQueue + 8500 
12  CoreFoundation                  0x183c7b044 __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__ + 24 
13  CoreFoundation                  0x183c7a3a0 __CFRunLoopDoSources0 + 256 
14  CoreFoundation                  0x183c78638 __CFRunLoopRun + 632 
15  CoreFoundation                  0x183bb96d0 CFRunLoopRunSpecific + 452 
16  GraphicsServices                0x189845c0c GSEventRunModal + 168 
17  UIKit                           0x186ceafdc UIApplicationMain + 1156 
18  TestNotification                0x100029990 main (main.m:16) 
19  libdyld.dylib                   0x190b63aa0 start + 4 

Thread 0 Crashed: 
0   libsystem_kernel.dylib          0x0000000190c5e58c __pthread_kill + 8 
1   libsystem_c.dylib               0x0000000190bf2804 abort + 108 
2   libc++abi.dylib                 0x000000018fe18990 abort_message + 84 
3   libc++abi.dylib                 0x000000018fe35c28 default_terminate_handler() + 296 
4   libobjc.A.dylib                 0x00000001905704d0 _objc_terminate() + 124 
5   libc++abi.dylib                 0x000000018fe33164 std::__terminate(void (*)()) + 12 
6   libc++abi.dylib                 0x000000018fe32d38 __cxa_rethrow + 140 
7   libobjc.A.dylib                 0x00000001905703a4 objc_exception_rethrow + 40 
8   CoreFoundation                  0x0000000183bb9748 CFRunLoopRunSpecific + 572 
9   GraphicsServices                0x0000000189845c08 GSEventRunModal + 164 
10  UIKit                           0x0000000186ceafd8 UIApplicationMain + 1152 
11  TestNotification                0x000000010002998c main (main.m:16) 
12  libdyld.dylib                   0x0000000190b63a9c start + 0

line3可以很清楚地看出，崩溃发生在MainViewController的42行，关于dSYM的介绍，详见这篇文章：
[crash log](http://www.raywenderlich.com/33669/overview-of-ios-crash-reporting-tools-part-1)

所以最好的办法，还是使用第三方框架。以Crashlytics为例，它会要求在xcode的编译过程中集成一个脚本，这个脚本的作用大致上是把二进制包和dSYM发送到Crashlytics的网站上，这样当Crashlytics拿到crash log以后，也就可以进行完全的符号化了