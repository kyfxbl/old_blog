title: iOS7中不支持自定义UIAlertView的样式
date: 2013-12-31 15:06
categories: iOS
---
今天遇到一个需求，需要让弹出的UIAlertView尺寸大一点，并且文字左对齐
<!--more-->

本来以为很简单，在网上一下就搜到了这段代码：

```
-(void) willPresentAlertView:(UIAlertView *)alertView{

    // 显示备份结果统计的对话框
    if(alertView.tag == ALERT_TAG_LOGOUT_DONE || alertView.tag == ALERT_TAG_BACKUP_DONE){

        for(UIView *subView in alertView.subviews){
            if([subView isKindOfClass:[UILabel class]]){
                UILabel* label = (UILabel*)subView;
                label.frame = CGRectMake(212, 234, 600, 300);
                label.textAlignment = NSTextAlignmentLeft;
            }
        }
    }
}
```
总的思路，就是在delegate的willPresentAlertView:方法里，拿到AlertView里的UILabel，然后设置其frame和textAlignment属性

代码看起来很有道理，不过只能作用在ios5和ios6下，在ios7里，这段代码是无效的！

[alertview in ios7](http://stackoverflow.com/questions/19027083/align-message-in-uialertview-to-left-in-ios-7)

所以，如果需要自定义AlertView的UI，只能通过子类来实现，或者使用这个现成的第三方组件：

[CustomIOS7AlertView](https://github.com/wimagguc/ios-custom-alertview)