title: 用storyboard开发iOS的体会
date: 2013-12-05 16:24
categories: iOS 
---
刚接触iOS开发不到10天，只写了一些后台的逻辑代码，对Objective-C和cocoa touch的API有点感觉了。不过今天开始尝试用storyboard开发UI，还是不太习惯，本文简单记录一下
<!--more-->

# 总的理解

iOS中一个页面称为一个scene，一般就对应一个ViewController，或者一个xib文件。而storyboard则是把应用中的scene都串联起来。一个ViewController下面一般只会有一个view的层次结构，构成了应用的UI

# 自动创建View

我不知道以前开发iOS是怎么做的，可能也需要写代码来初始化ViewController吧。不过今天用storyboard来开发，似乎不需要，storyboard会隐式地创建和初始化配置的ViewController，以及ViewController下面挂的所有View组件

所以一般不需要写这样的代码：

```
- (void)viewDidLoad    
{    
    [super viewDidLoad];    
    self.webView = [[UIWebView alloc] initWithFrame:CGRectMake(0, 0, 320, 480)];    
    NSURLRequest *request =[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.baidu.com"]];    
    [self.view addSubview: self.webView];    
    [self.webView loadRequest:request];    
}
```
总的来说，UI的开发可以在storyboard里通过图形化界面来完成。不过在需要的时候，也可以编码来实现，不过UI的API我还不太熟

# 获取View的引用

虽然storyboard自动创建了View组件，但是代码里还是需要拿到它们的引用才可以。在android里经常可以看到这样的代码：

```
mobileNumberLayout = (LinearLayout) findViewById(R.id.mobileNumberLayout);
sendValidateCodeTv = (TextView) findViewById(R.id.sendValidateCode);
```

主要就是findViewById这个API，在ios开发里不需要这样调用，View在Controller里的引用有个术语称为IBOutlet，只要在storyboard里拉线，就可以直接引用了：

```
// 刷新进度条和文本
- (void) refreshProgressTo: (float)stage WithContent:(NSString*)content
{
    [self.loadingLabel setText:content];
    [self.progress setProgress:stage animated: YES];
}
```

这点我觉得还是挺方便的，要善用storyboard里的connections inspector面板，里面会显示有哪些Outlet，Action，Segue等，很方便

# 用Action响应UI事件

最后是怎么响应UI事件，比如按钮按下，文本框内容变化等

在android里比较类似html里的写法，有onclick之类的。在iOS里也差不多，主要是target-action模型。View Controller称为target，然后可以设置各种Action。而View会发出Event，可以配置Event会触发target上的哪个Action

Action的函数必须是这种格式：

```
-(IBAction) doSomething:(id*) sender
{
    NSLog(@"hahaha",nil);
}
```

然后在storyboard里就会识别出这是个Action，可以通过拉线跟View的Event关联起来

如果不用storyboard的话，也一样是这个模式，只不过需要自己写代码注册Action和Event的关联关系，方法是在UIControl中定义的：

```
-(void)addTarget:(id)target action:(SEL) forControlEvents:(UIControlEvents)controlEvents
```

举例：

```
[self.btnCooking addTarget:self action:@selector(pressCooking:) forControlEvents:UIControlEventTouchUpInside];
```

上面这段代码，当btnCooking发出TouchUpInside事件时，就会调用Controller上的pressCooking函数（@selector在OC中就是方法的意思）

# 感想

既然能拿到View的引用，又能设置Action，那基本就什么都能做了。有些功能用storyboard来实现还是不错的，特别是auto layout似乎挺多人都说挺好。不过也要熟悉iOS UI相关的API，在必要的时候才能编码来实现