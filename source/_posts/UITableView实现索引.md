title: UITableView实现索引
date: 2014-06-19 22:17
categories: iOS 
---
UITableView实现字母索引
<!--more-->

效果图类似这样，普通的通讯录样式

![](http://img.blog.csdn.net/20140619222150703)

在网上找到很多帖子，都没成功，最后还是官方的文档靠谱：

[TableView PG](https://developer.apple.com/library/ios/documentation/userexperience/conceptual/tableview_iphone/CreateConfigureTableView/CreateConfigureTableView.html#//apple_ref/doc/uid/TP40007451-CH6-SW30)

其实最关键的，就是数据源。需要把原始的数据结构，组织成二维数组，第一层数组用于分section，第二层数组是每个section中的元素。主要代码如下（清晰起见，省略无关代码）：

每一行的数据结构：

```
@interface Member : NSObject

@property(nonatomic,copy) NSString *pk;
@property(nonatomic,copy) NSString *name;
@property NSInteger sectionNumber;

@end
```

首先是从数据源中读取原始数据，这时还没有分组：
```
NSMutableArray *membersTemp = [NSMutableArray array];

while(){
    Member *member = [[Member alloc] initWithPk:pk Name:name];
    [membersTemp addObject:member];
}
```

然后给每个Memer分配SectionNumber，这里用到了UILocalizedIndexedCollation类，不需要自己处理索引分组。网上很多例子都用到外部的分组方法，其实是不需要的：

```
UILocalizedIndexedCollation *collation = [UILocalizedIndexedCollation currentCollation];

for(Member *member in membersTemp) {
    NSInteger sect = [collation sectionForObject:member collationStringSelector:@selector(name)];
    member.sectionNumber = sect;
}
```

然后创建了27个数组，分别是A-Z和\#的索引，数组只是初始化，还没有把Member放进去：

```
NSInteger highSection = [[collation sectionTitles] count];
NSMutableArray *sectionArrays = [NSMutableArray arrayWithCapacity:highSection];
for(int i = 0; i < highSection; i++) {
    NSMutableArray *sectionArray = [NSMutableArray arrayWithCapacity:1];
    [sectionArrays addObject:sectionArray];
}
```

然后把Member放入合适的数组：

```
for(Member *member in membersTemp) {
    [(NSMutableArray*)[sectionArrays objectAtIndex:member.sectionNumber] addObject:member];
}
```

最后，把数组排序，放到真正的members里，这样members就是一个二维数组：

```
for(NSMutableArray *sectionArray in sectionArrays) {
    NSArray *sortedSection = [collation sortedArrayFromArray:sectionArray collationStringSelector:@selector(name)];
    [members addObject:sortedSection];
}
```

经过上面的代码，members就是已经组织好的二维数组。接下来要实现TableViewDataSource的代理方法：

```
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView
{
    return [members count];// 多少个section
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    return [[members objectAtIndex:section] count];// section的row数，会调用多次
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{    
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:[UITableViewCell reuseIdentifier]];
    if(!cell) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:[UITableViewCell reuseIdentifier]];
    }

    Member *member = [[members objectAtIndex:indexPath.section] objectAtIndex:indexPath.row];// 每一行
    cell.textLabel.text = member.name;
    return cell;
}

- (NSArray *)sectionIndexTitlesForTableView:(UITableView *)tableView
{
    return [[UILocalizedIndexedCollation currentCollation] sectionIndexTitles];// A-Z, #
}

- (NSString *)tableView:(UITableView *)tableView titleForHeaderInSection:(NSInteger)section{

    if ([[members objectAtIndex:section] count] > 0) {
        return [[[UILocalizedIndexedCollation currentCollation] sectionTitles] objectAtIndex:section];// 空的数组，没有title
    }
    return nil;
}

- (NSInteger)tableView:(UITableView *)tableView sectionForSectionIndexTitle:(NSString *)title atIndex:(NSInteger)index
{
    return [[UILocalizedIndexedCollation currentCollation] sectionForSectionIndexTitleAtIndex:index];// 点击索引导航
}
```

关于这个问题，官方文档比网上任何文档都清楚