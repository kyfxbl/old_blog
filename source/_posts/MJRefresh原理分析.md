title: MJRefresh原理分析
date: 2015-07-26 16:25:56
categories: iOS
---
MJRefresh是流行的下拉刷新控件，前段时间为了修复一个BUG，读了它的源码，本文总结一下实现的原理
<!--more-->

# 下拉刷新的基本原理

大部分的下拉刷新控件，都是用contentInset实现的。默认情况下，如果一个UIScrollView的左上角在导航栏的正下方，那么它的contentInset是64，而contentOffset是-64。继续下拉的话，contentOffset就会越来越小，如果上滑，contentOffset就会增大，直到左上角达到屏幕的左上角时，contentOffset刚好为0

默认情况下，如果下拉一个UIScrollView，在松手之后，会弹回初始的位置（导航栏下方）。而大部分的下拉刷新控件，都是将自己放在UIScrollView的上方，起始y设置成负数，所以平时不会显示出来，只有下拉的时候才会出现，放开又会弹回去。然后在loading的时候，临时把contentInset增大，相当于把UIScrollView往下挤，于是下拉刷新的控件就会显示出来，然后刷新完成之后，再把contentInset改回原来的值，实现回弹的效果

基本上，MJRefresh也是这么实现的

# 创建下拉刷新控件实例

从创建实例的代码开始：
```
MJRefreshNormalHeader *header = [MJRefreshNormalHeader headerWithRefreshingBlock:^{
    
    [myController loadCollectionDataNeedReset:YES withBlock:^{
        [self.header endRefreshing];
        [self reloadData];
    }];
}];
```
调用的是一个工厂方法headerWithRefreshingBlock，这个方法定义在各种header控件的基类MJRefreshHeader里：
```
+ (instancetype)headerWithRefreshingBlock:(MJRefreshComponentRefreshingBlock)refreshingBlock
{
    MJRefreshHeader *cmp = [[self alloc] init];
    cmp.refreshingBlock = refreshingBlock;
    return cmp;
}
```
然后会调用init方法，由于MJRefreshHeader里并没有定义init方法，而它的基类MJRefreshComponent里定义了，所以会进入到基类的初始化方法里：
```
- (instancetype)initWithFrame:(CGRect)frame
{
    if (self = [super initWithFrame:frame]) {
        // 准备工作
        [self prepare];
        
        // 默认是普通状态
        self.state = MJRefreshStateIdle;
    }
    return self;
}
```
这里的关键是prepare方法，这个方法是___第一个扩展点___，具体的header（包括库提供的原生header，和用户自定义的header）有哪些属性，样式是怎么样，都是在这个方法里实现的。每个子类的prepare方法，都会调用父类的prepare方法。所以在扩展的时候，公共的属性写在父类的prepare方法里，特有的属性写在子类的prepare方法里。比如，我们看一下MJRefreshStateHeader的：
```
- (void)prepare
{
    [super prepare];
    
    // 初始化文字
    [self setTitle:MJRefreshHeaderIdleText forState:MJRefreshStateIdle];
    [self setTitle:MJRefreshHeaderPullingText forState:MJRefreshStatePulling];
    [self setTitle:MJRefreshHeaderRefreshingText forState:MJRefreshStateRefreshing];
}
```
总之，调用headerWithRefreshingBlock方法以后，就得到了一个UIView的实例，也就是下拉刷新的控件。但是现在它还没有挂到任何superview上，也没有任何行为

# 将下拉刷新控件，挂到UIScrollView上

接下来的调用：
```
self.header = header;
```
这是利用了UIScrollView+MJRefresh里的一个category，为UIScrollView增加了属性header和footer。这里用到了关联对象的技巧（AssociatedObject），因为category通常情况下是不能直接添加实例变量的
```
- (void)setHeader:(MJRefreshHeader *)header
{
    if (header != self.header) {
        // 删除旧的，添加新的
        [self.header removeFromSuperview];
        [self addSubview:header];
        
        // 存储新的
        [self willChangeValueForKey:@"header"]; // KVO
        objc_setAssociatedObject(self, &MJRefreshHeaderKey,
                                 header, OBJC_ASSOCIATION_ASSIGN);
        [self didChangeValueForKey:@"header"]; // KVO
    }
}
```
通过上面的代码，把header添加到了UIScrollView的subviews里，并保留了一个引用。但是这个header的frame还没有确定，也没有任何行为

# 设置header的位置和侦听行为

由于上面执行了addSubview，接下来就会进入header的生命周期方法willMoveToSuperview，这个方法是在公共的基类MJRefreshComponent里实现的。因为这是基础的行为，所以写在公共的基类里，所有的子类都能共享：
```
- (void)willMoveToSuperview:(UIView *)newSuperview
{
    [super willMoveToSuperview:newSuperview];
    
    // 如果不是UIScrollView，不做任何事情
    if (newSuperview && ![newSuperview isKindOfClass:[UIScrollView class]]) return;
    
    // 旧的父控件移除监听
    [self removeObservers];
    
    if (newSuperview) { // 新的父控件
        // 设置宽度
        self.mj_w = newSuperview.mj_w;
        // 设置位置
        self.mj_x = 0;
        
        // 记录UIScrollView
        _scrollView = (UIScrollView *)newSuperview;
        // 设置永远支持垂直弹簧效果
        _scrollView.alwaysBounceVertical = YES;
        // 记录UIScrollView最开始的contentInset
        _scrollViewOriginalInset = self.scrollView.contentInset;
        
        // 添加监听
        [self addObservers];
    }
}
```
这里关键是设置了alwaysBounceVertical，这样才能确保UIScrollView可以下拉，否则需要处理contentSize才能拉得动，就麻烦了很多。此外这里令header也持有UIScrollView的引用，后续可以从上面取到各种属性

然后是添加监听的方法addObservers，这里主要是用了KVO的技巧：
```
- (void)addObservers
{
    NSKeyValueObservingOptions options = NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld;
    [self.scrollView addObserver:self forKeyPath:MJRefreshKeyPathContentOffset options:options context:nil];
    [self.scrollView addObserver:self forKeyPath:MJRefreshKeyPathContentSize options:options context:nil];
    self.pan = self.scrollView.panGestureRecognizer;
    [self.pan addObserver:self forKeyPath:MJRefreshKeyPathPanState options:options context:nil];
}
```
```
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
{
    // 遇到这些情况就直接返回
    if (!self.userInteractionEnabled || self.hidden) return;
    
    if ([keyPath isEqualToString:MJRefreshKeyPathContentOffset]) {
        [self scrollViewContentOffsetDidChange:change];
    } else if ([keyPath isEqualToString:MJRefreshKeyPathContentSize]) {
        [self scrollViewContentSizeDidChange:change];
    } else if ([keyPath isEqualToString:MJRefreshKeyPathContentInset]) {
        [self scrollViewContentInsetDidChange:change];
    } else if ([keyPath isEqualToString:MJRefreshKeyPathPanState]) {
        [self scrollViewPanStateDidChange:change];
    }
}
```
这里侦听了3个key的变化，UIScrollView的contentOffset和contentSize，以及滑动手势的状态。然后在每个value发生变化的时候，调用几个didChange方法。这些didChange方法都是hook，是___第二个扩展点___，实际上都是由子类来实现的

接下来会进入生命周期方法layoutSubviews：
```
- (void)layoutSubviews
{
    [super layoutSubviews];
    
    [self placeSubviews];
}
```
这里的placeSubviews就是header应该怎么摆，是___第三个扩展点___，把header的origin.y设置成负值，就是在MJRefreshHeader的这个方法里实现的：
```
- (void)placeSubviews
{
    [super placeSubviews];
    
    // 设置y值(当自己的高度发生改变了，肯定要重新调整Y值，所以放到placeSubviews方法中设置y值)
    self.mj_y = - self.mj_h;
}
```
每个子类的placeSubviews方法，都应该先调用父类的这个方法

通过上述的代码，确定了下拉刷新控件的位置，以及其中每个subview的位置。并且侦听了UIScrollView的contentOffset和contentSize变化

# 下拉时的实际行为

下拉会导致contentOffset变化，由于前面已经添加了KVO侦听，所以会执行scrollViewContentOffsetDidChange方法：
```
- (void)scrollViewContentOffsetDidChange:(NSDictionary *)change
{
    [super scrollViewContentOffsetDidChange:change];
    
    // 在刷新的refreshing状态
    if (self.state == MJRefreshStateRefreshing) {
        // sectionheader停留解决
        return;
    }
    
    // 跳转到下一个控制器时，contentInset可能会变
    _scrollViewOriginalInset = self.scrollView.contentInset;
    
    // 当前的contentOffset
    CGFloat offsetY = self.scrollView.mj_offsetY;
    // 头部控件刚好出现的offsetY
    CGFloat happenOffsetY = - self.scrollViewOriginalInset.top;
    
    // 如果是向上滚动到看不见头部控件，直接返回
    if (offsetY >= happenOffsetY) return;
    
    // 普通 和 即将刷新 的临界点
    CGFloat normal2pullingOffsetY = happenOffsetY - self.mj_h;
    CGFloat pullingPercent = (happenOffsetY - offsetY) / self.mj_h;
    if (self.scrollView.isDragging) { // 如果正在拖拽
        self.pullingPercent = pullingPercent;
        if (self.state == MJRefreshStateIdle && offsetY < normal2pullingOffsetY) {
            // 转为即将刷新状态
            self.state = MJRefreshStatePulling;
        } else if (self.state == MJRefreshStatePulling && offsetY >= normal2pullingOffsetY) {
            // 转为普通状态
            self.state = MJRefreshStateIdle;
        }
    } else if (self.state == MJRefreshStatePulling) {// 即将刷新 && 手松开
        // 开始刷新
        [self beginRefreshing];
    } else if (pullingPercent < 1) {
        self.pullingPercent = pullingPercent;
    }
}
```
这段代码比较长，主要是判断offset变化是否达到了临界值，以及当前的手势，切换header的state状态，然后根据state状态变化，驱动不同的行为：
```
- (void)setState:(MJRefreshState)state
{
    MJRefreshCheckState
    
    // 根据状态做事情
    if (state == MJRefreshStateIdle) {
        if (oldState != MJRefreshStateRefreshing) return;
        
        // 保存刷新时间
        [[NSUserDefaults standardUserDefaults] setObject:[NSDate date] forKey:self.lastUpdatedTimeKey];
        [[NSUserDefaults standardUserDefaults] synchronize];
        
        // 恢复inset和offset
        [UIView animateWithDuration:MJRefreshSlowAnimationDuration animations:^{
            self.scrollView.mj_insetT -= self.mj_h;
            
            // 自动调整透明度
            if (self.isAutoChangeAlpha) self.alpha = 0.0;
        } completion:^(BOOL finished) {
            self.pullingPercent = 0.0;
        }];
    } else if (state == MJRefreshStateRefreshing) {
        [UIView animateWithDuration:MJRefreshFastAnimationDuration animations:^{
            // 增加滚动区域
            CGFloat top = self.scrollViewOriginalInset.top + self.mj_h;
            self.scrollView.mj_insetT = top;
            
            // 设置滚动位置
            self.scrollView.mj_offsetY = - top;
        } completion:^(BOOL finished) {
            [self executeRefreshingCallback];
        }];
    }
}
```
setState方法是___第四个扩展点___，这里的MJRefreshCheckState是个宏，也调用了父类的setState的方法。下拉的时候临时增大contentInset，令header保留在屏幕上，然后调用callback block；结束之后还原contentInset