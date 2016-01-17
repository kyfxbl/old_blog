title: iOS接收notification重复
date: 2014-01-07 14:36
categories: iOS 
---
今天修改了一个困扰了好几天的BUG，本文记录一下
<!--more-->

现象是，点击1次按钮，有时会弹出2次对话框。一开始以为是偶现的BUG，很难定位。今天才发现，在页面第一次加载时，不会出现，而从第二次加载开始，必定出现。最后调试跟踪到这段代码：

```
- (void)cloudBackup:(CDVInvokedUrlCommand*)command
{
    [[NSNotificationCenter defaultCenter] postNotificationName:BACKUP_START object:self userInfo:nil];

    NSMutableDictionary* data = [NSMutableDictionary dictionaryWithCapacity:5];
    CDVPluginResult* pluginResult = [CDVPluginResult resultWithStatus:CDVCommandStatus_OK messageAsDictionary:data];
    [self.commandDelegate sendPluginResult:pluginResult callbackId:command.callbackId];
}
```

```
-(void) listenBackupStartEvent:(NSNotification*) notification
```

前者是响应按钮的点击，然后发送一个Notification，后者侦听此Notification，并进行实际处理。断点发现，第一个方法只调用一次，但是第2个方法却调用了2次。因此可以确定，是因为ViewController类重复收到了通知

确定了这点以后，再定位就很简单了。因为将注册侦听器的代码：

```
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(listenBackupStartEvent:) name:BACKUP_START object:nil];
```
写在viewDidLoad()方法里，但是在退出此页面时，没有注销侦听器，所以当下次重新加载此页面，就会重复注册侦听器，最终造成此BUG

结论：

1、注册侦听器的代码，最好不要写在vieDidLoad()里，放在init()里似乎更好

2、一旦注册过侦听器，一定要仔细考虑注销侦听器的时机，避免重复收到通知