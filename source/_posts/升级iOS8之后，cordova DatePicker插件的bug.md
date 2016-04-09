title: 升级iOS8之后，cordova DatePicker插件的bug
date: 2014-10-18 11:03
categories: iOS 
---
升级到iOS8之后，通过cordova的DatePicker插件弹出UIDatePicker控件，会导致应用crash
<!--more-->

错误信息：

Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: 'UITableView dataSource is not set'

查看DatePicker的源码：

```
if(!self.datePicker){
            self.datePicker = [self createDatePicker:options frame:frame];
            [self.datePicker addTarget:self action:@selector(dateChangedAction:) forControlEvents:UIControlEventValueChanged];
        }
```
它的目的是不重复创建UIDatePicker，只在第一次弹出时创建一个UIDatePicker的实例，然后就往新的UIView上挂，其结果就是多个UIView共享同一个UIDatePicker实例。这个行为在iOS7下成立，但是在升级到iOS8之后就会导致crash

所以将这段代码改成：

```
// in iOS8, UIDatePicker couldn't be shared in multi UIViews, it will cause crash. so create new UIDatePicker instance every time
    if ([[[UIDevice currentDevice] systemVersion] floatValue] >= 8.0) {

        self.datePicker = [self createDatePicker:options frame:frame];
        [self.datePicker addTarget:self action:@selector(dateChangedAction:) forControlEvents:UIControlEventValueChanged];

    }else{

        if(!self.datePicker){
            self.datePicker = [self createDatePicker:options frame:frame];
            [self.datePicker addTarget:self action:@selector(dateChangedAction:) forControlEvents:UIControlEventValueChanged];
        }
    }
```

另外，iOS是不建议UIView共享实例的，包括多个UIView共享控件，把一个UIView挂到不同的UIViewController下，或者把UIView设置成单例，都是不好的做法。自己写代码的时候也要注意避免