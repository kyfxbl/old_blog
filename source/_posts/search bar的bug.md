title: search bar的bug
date: 2014-07-18 21:13
categories: iOS 
---
iOS中UISearchBar的delegate方法被重复调用的bug
<!--more-->

从ios4开始，UISearchBar就有一个BUG：当点击搜索输入框里的小叉号时，delegate方法会被调用2次：

```
-(void) searchBar:(UISearchBar*)searchBar textDidChange:(NSString*)searchText
```

在SO上找了一番，没有看到更好的办法，最后我是通过加锁的方法来规避：
```
{
    BOOL searchLock;// to handle ios search bar bug
}
```

```
-(void) searchBar:(UISearchBar*)searchBar textDidChange:(NSString*)searchText
{
    if(searchLock){
        return;
    }

    searchLock = YES;

    // do job

    searchLock = NO;
}
```

另外，UISearchBar可以设置一个取消button，但是一直显示着比较难看。所以我设置成默认不展示，当开始输入搜索条件时才展示，输入结束之后再隐藏掉
```
- (void)searchBarTextDidBeginEditing:(UISearchBar *)searchBar
{
    ContactView* myView = (ContactView*)self.view;
    myView.search.showsCancelButton = YES;
}
```
```
- (void)searchBarCancelButtonClicked:(UISearchBar *) searchBar
{
    ContactView* myView = (ContactView*)self.view;
    myView.search.showsCancelButton = NO;
    [myView.search resignFirstResponder];
}
```