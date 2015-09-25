title: objective-c浮点数相除
date: 2013-11-30 17:09
categories: iOS  
---
开发iOS应用时，如果涉及到浮点数相除，要注意变量的类型
<!--more-->

今天在实现一个特性，根据完成的步骤分阶段刷新progress bar。结果发现只有最后一步会刷新进度条，前面完成的步骤都无效

代码是：
```
- (void) notifyStepDone:(NSString*) content
{
    doneSteps ++;
    float currentProgress = doneSteps/maxSteps;
    [_delegate refreshProgressTo:currentProgress WithContent:content];
}
```

调试以后发现，原来在前面的步骤里，currentProgress的值总是0。才想到doneSteps和maxSteps的类型都声明为int，所以1/4，2/4的结果都是0，而不是预期的0.25和0.5