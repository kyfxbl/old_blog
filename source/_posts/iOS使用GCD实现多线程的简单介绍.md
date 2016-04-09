title: iOS使用GCD实现多线程的简单介绍
date: 2013-12-20 16:10
categories: iOS 
---
应用有一个简单的需求，即在后台做文件下载等一系列操作，过程中在主页面刷新进度条，全部完成后弹出对话框提示用户
<!--more-->

这个需求显然必须用到多线程，在网上搜索了一番，在ios里实现多线程主要有3种方式：

1、NSThread，以及在此基础上封装的performSelector方法，如performSelectorInBackground和performSelectorOnMainThread等简便方法

2、NSOperation，这个方式我从来没用过，不了解

3、GCD，今天我改成了这个实现方式，本文主要总结关于GCD的用法

# NSThread的缺点

本来我是用NSThread + Notification来实现线程切换，不过随着UI变得复杂，代码也越来越不清晰了。主要是NSThread有以下几个缺点：

1、经常需要配合Notification一起使用，如果流程比较复杂，系统中会有很多事件，理解起来很不直观

2、performSelector方法的参数，只能接受SEL，不能传递block进去，所以系统中需要很多“跳板方法”

3、侦听事件的方法，只能接受一个参数即notification，然后取出userInfo，类型也限定为NSDictionary，复杂交互时，传参很不方便

# GCD

相比之下，GCD可以克服上面的几个问题，不需要配合notification就能工作，而且block可以形成闭包，传参也很方便

更重要的是，GCD是apple官方推荐的方式，所以很多第三方组件都是用GCD来实现的，应用使用GCD有时候就可以和第三方组件很好地配合

# 示例代码

我今天才刚开始接触，对API也了解得不全，主要有4个函数比较常用，可以满足一般的场景了（GCD的API是C风格，不是objective-c风格）

```
dispatch_async

dispatch_sync
```

就是开启新线程的API，前者是立刻返回，不阻塞当前线程；后者则是同步调用，会阻塞当前的线程，所以在UI Thread里，要小心调用后者

这2个函数都接受2个参数，第1个是线程，第2个是block。block就不用多解释了，为了传第1个参数，就用到下面2个API

```
dispatch_get_global_queue

dispatch_get_main_queue
```

前者是整个进程（应用）共享的子线程，后者就是UI Thread，基本上这4个API，就足够应付一般的需求了

下面贴一段示例代码：

```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^(void){

        BOOL needBackup = [self.backupDelegate checkNeedBackup];// 跑在子线程

        dispatch_async(dispatch_get_main_queue(), ^(void){
            if(needBackup){
                UIAlertView *confirmBackupAlert = [[UIAlertView alloc] initWithTitle:nil message:NSLocalizedString(@"backup_confirm_alert_view_message", @"") delegate:mainViewDelegate cancelButtonTitle:NSLocalizedString(@"button_cancel", @"") otherButtonTitles:NSLocalizedString(@"button_confirm", @""), nil];
                confirmBackupAlert.tag = ALERT_TAG_CONFIRM_BACKUP;
                [confirmBackupAlert show];
            }else{
                UIAlertView *noNeedBackupAlert = [[UIAlertView alloc] initWithTitle:nil message:NSLocalizedString(@"backup_no_need", @"") delegate:mainViewDelegate cancelButtonTitle:NSLocalizedString(@"button_iknow", @"") otherButtonTitles:nil];
                noNeedBackupAlert.tag = ALERT_TAG_BACKUP_NO_NEED;
                [noNeedBackupAlert show];
            }
        });

    });
```
在子线程做费时操作，比如HTTP请求和数据库访问等，然后切换回主线程，弹出AlertView

# 好链接

下面2个链接都很不错：

[ios中消息的传递机制](http://beyondvincent.com/blog/2013/12/14/124-communication-patterns/)

[51CTO多线程专题](http://mobile.51cto.com/iphone-403490.htm)