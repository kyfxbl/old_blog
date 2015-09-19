title: 单独使用NavigationBar
date: 2014-06-24 19:20
categories: iOS 
---
大部分情况下，NavigationBar都是和UINavigationController一起使用的。本文总结一下单独使用NavigationBar的方法
<!--more-->

跟UINavigationController一起用的时候，设置比较简单，直接通过ViewController的self.navigationItem属性就可以设置：

```
UIButton *switchShop = [[UIButton alloc] initWithFrame:CGRectMake(0, 0, 20, 20)];
[switchShop setBackgroundImage:[UIImage imageNamed:@"switch_shop"] forState:UIControlStateNormal];
[switchShop addTarget:self action:@selector(switchButtonTapped) forControlEvents:UIControlEventTouchUpInside];
self.navigationItem.rightBarButtonItem = [[UIBarButtonItem alloc] initWithCustomView:switchShop];
```

但是在特殊情况下，NavigationBar也可以单独使用。这样就需要手工创建NavigationBar，NavigationItem，并手工加到view上：

```
-(void) loadView
{
    RegisterView *view = [[RegisterView alloc] initWithController:self];
    self.view = view;

    UINavigationBar *navigationBar = [[UINavigationBar alloc] initWithFrame:CGRectMake(0, 0, 320, 66)];

    UINavigationItem *navigationItem = [[UINavigationItem alloc] initWithTitle:nil];

    if([self.type isEqualToString:@"register"]){
        navigationItem.title = @"注册";
    }else{
        navigationItem.title = @"重置密码";
    }

    UIBarButtonItem *backButton = [[UIBarButtonItem alloc] initWithTitle:@"关闭" style:UIBarButtonItemStyleBordered target:self action:@selector(back)];
    navigationItem.leftBarButtonItem = backButton;

    [navigationBar pushNavigationItem:navigationItem animated:NO];

    [self.view addSubview:navigationBar];
}
```