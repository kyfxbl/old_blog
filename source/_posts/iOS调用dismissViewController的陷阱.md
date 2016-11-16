title: iOS调用dismissViewController的陷阱
date: 2014-12-17 23:30
categories: iOS 
---
我们的APP从启动到进入主页面，是通过presentViewController构造了一个ViewController序列，类似于首页 登陆页 启动加载页 主页面
<!--more-->

其中，在启动加载页的viewDidAppear方法里做了很多逻辑处理：

```
-(void) viewDidAppear:(BOOL)animated{

    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^(void){

        clientInfo = [YLSClientInfo new];

        if([clientInfo needInit]){
            [self mkdirAndDatabaseFile];
        }else{
            [self refreshVersion:[clientInfo currentVersion]];
        }

       // 各种处理逻辑
    });
}
```

然后进入主页面之后，如果用户退出登陆，就需要回到首页，所以会在首页上调用dismissViewController方法。原先的代码类似这样：

```
UIViewController *origin = self.presentingViewController.presentingViewController;
if([origin isMemberOfClass:[YLSLoginViewController class]]){
    origin = self.presentingViewController.presentingViewController.presentingViewController;
}
[origin dismissViewControllerAnimated:NO completion:nil];
```

预期的结果是，直接回到首页，然后触发首页的viewDidAppear方法。实际上通过观察console warning才发现，中间启动加载页的viewDidAppear方法也被调用了。登陆页由于没有写viewDidAppear方法，所以没有发现，但我猜测如果有的话，也一样会被调用。似乎ViewController是按照顺序一个接一个出栈的，所以每一个“之前的”ViewController的viewDidAppear方法应该都会被触发

查了一下API，又上stackoverflow搜索了半天，似乎没有办法阻止这个默认行为。所以最后我的解决办法是在中间的Controller上加了标记：

```
-(void) viewDidAppear:(BOOL)animated{

    // 如果是由于调用了dismiss而触发了此方法，不进行初始化
    if(self.isDismissing){
        return;
    }

   // 初始化加载逻辑
}
```

```
YLSBootstrapViewController *bootstrapController = (YLSBootstrapViewController*)self.presentingViewController;
bootstrapController.isDismissing = YES;

UIViewController *origin = self.presentingViewController.presentingViewController;
if([origin isMemberOfClass:[YLSLoginViewController class]]){
    origin = self.presentingViewController.presentingViewController.presentingViewController;
}
[origin dismissViewControllerAnimated:NO completion:nil];
```