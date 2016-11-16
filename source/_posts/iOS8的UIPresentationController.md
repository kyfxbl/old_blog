title: iOS8的UIPresentationController
date: 2014-10-25 16:23
categories: iOS 
---
从iOS8开始，controller之间的跳转特效，需要用新的API UIPresentationController来实现。
<!--more-->

比如希望实现这样一个特效：显示一个模态窗口，大小和位置是自定义的，遮罩在原来的页面上。在iOS8之前，可以在viewWillAppear里设置superview的frame：

```
- (void)presentModal:(NSDictionary*)result
{
    YLSCheckoutSignatureController *controller = [[YLSCheckoutSignatureController alloc] initWithModel:result];

    if ([[[UIDevice currentDevice] systemVersion] floatValue] >= 8.0) {
        controller.modalPresentationStyle = UIModalPresentationCustom;
    }else{
        controller.modalTransitionStyle = UIModalTransitionStyleCrossDissolve;
        controller.modalPresentationStyle = UIModalPresentationFormSheet;
    }

    [self presentViewController:controller animated:YES completion:nil];
}
```
```
-(void) viewWillAppear:(BOOL)animated
{
    // in iOS8, handle by UIPresentationController
    if ([[[UIDevice currentDevice] systemVersion] floatValue] >= 8.0) {
        return;
    }

    self.view.superview.layer.cornerRadius = 10;
    self.view.superview.layer.borderColor = [UIColor darkGrayColor].CGColor;
    self.view.superview.clipsToBounds = YES;
    self.view.superview.frame = CGRectMake(62, 114, 900, 540);
}
```
但是以上的代码，在iOS8里就不再生效了，要用UIPresentationController来实现

首先明确一点，从ControllerA跳转到B，B的样式和跳转特效，还是由B来控制的。只不过以前是直接在Controller的生命周期方法里操作，而现在有专门的API来完成而已。这种设计也是合理的，否则如果从A可以跳转到B和C，但是样式和特效不一样，就只能通过在A里面设置实例变量来区分了，容易出错也很别扭。所以把跳转的行为由目标Controller来控制是很合理的

不过这组API的文档不太全，后续SDK升级可能会逐渐完善。以下介绍实现步骤：

# 目标Controller实现特定protocol

首先目标Controller要实现特定的协议，创建一个UIPresentationController

```
@interface YLSCheckoutSignatureController  : UIViewController<UIScrollViewDelegate, UIViewControllerTransitioningDelegate>
```
```
self.transitioningDelegate = self;
```
```
- (UIPresentationController *)presentationControllerForPresentedViewController:(UIViewController *)presented presentingViewController:(UIViewController *)presenting sourceViewController:(UIViewController *)source
{
    return [[YLSMainPresentationController alloc] initWithPresentedViewController:presented presentingViewController:presenting];
}
```
当条件满足时，iOS系统会调用这个方法，于是可以实例化自定义的UIPresentationController子类，定义跳转的样式和特效

# 自定义UIPresentationController

然后就要实现自定义的UIPresentationController，下面这段实例代码，实现居中展示一个自定义frame的模态页面，同时有半透明背景遮住原来的页面

```
@implementation YLSMainPresentationController

{
    UIView *dimmingView;
}

-(id) initWithPresentedViewController:(UIViewController *)presentedViewController presentingViewController:(UIViewController *)presentingViewController
{
    self = [super initWithPresentedViewController:presentedViewController presentingViewController:presentingViewController];
    if(self){

        dimmingView = [[UIView alloc] init];
        dimmingView.backgroundColor = [UIColor grayColor];
        dimmingView.alpha = 0.0;
    }
    return self;
}

- (void)presentationTransitionWillBegin
{
    dimmingView.frame = self.containerView.bounds;
    [self.containerView addSubview:dimmingView];
    [self.containerView addSubview:self.presentedView];

    id<UIViewControllerTransitionCoordinator> coordinator = self.presentingViewController.transitionCoordinator;

    [coordinator animateAlongsideTransition:^(id<UIViewControllerTransitionCoordinatorContext> context) {
        dimmingView.alpha = 0.5;
    } completion:nil];
}

- (void)presentationTransitionDidEnd:(BOOL)completed
{
    if(!completed){
        [dimmingView removeFromSuperview];
    }
}

- (void)dismissalTransitionWillBegin
{
    id<UIViewControllerTransitionCoordinator> coordinator = self.presentingViewController.transitionCoordinator;

    [coordinator animateAlongsideTransition:^(id<UIViewControllerTransitionCoordinatorContext> context) {
        dimmingView.alpha = 0.0;
    } completion:nil];
}

- (void)dismissalTransitionDidEnd:(BOOL)completed
{
    if(completed){
        [dimmingView removeFromSuperview];
    }
}

- (CGRect)frameOfPresentedViewInContainerView
{
    return CGRectMake(62.f, 114.f, 900.f, 540.f);
}

@end
```

代码确实比以前复杂了一点，但是其实每个生命周期方法都是比较明确的，开发者可控的粒度也更细了。比如设置presented frame，就有专门的方法，只要返回CGRect就可以了，还是比较方便的

# 原始的ViewController发起跳转动作

经过前面2步，当自定义跳转发生时，就可以很细致地控制样式和跳转行为。接下来就是由原始controller（presenting view controller）来发起跳转动作：

```
- (void)presentModal:(NSDictionary*)result
{
    YLSCheckoutSignatureController *controller = [[YLSCheckoutSignatureController alloc] initWithModel:result];

    if ([[[UIDevice currentDevice] systemVersion] floatValue] >= 8.0) {
        controller.modalPresentationStyle = UIModalPresentationCustom;
    }else{
        controller.modalTransitionStyle = UIModalTransitionStyleCrossDissolve;
        controller.modalPresentationStyle = UIModalPresentationFormSheet;
    }

    [self presentViewController:controller animated:YES completion:nil];
}
```
关键是设置modalPresentationStyle为UIModalPresentationCustom，然后当presentViewController方法调用时，iOS系统就会创建出UIPresentationController的实例，来控制跳转的行为