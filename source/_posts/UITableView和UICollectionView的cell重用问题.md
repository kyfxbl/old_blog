title: UITableView和UICollectionView的cell重用问题
date: 2015-05-02 15:45
categories: iOS 
---
APP的一个页面用到了自定义的UITableViewCell，由于iOS框架的cell重用机制，遇到了一个BUG，总结一下
<!--more-->

# 现象

自定义的UITableViewCell里有一个UIButton，点击这个button以后，需要改变cell的样式，包括换UILabel字体颜色，禁用该UIButton等。结果发现，点击按钮之后，不仅当前cell的字体颜色变了，还有另外几个cell的字体颜色也跟着变，而且是随机的

# 原因

后来想到，应该是由于iOS的cell重用机制造成的，原来的代码类似：

```
-(void) onButtonPressed
{
    label.textColor = [UIColor grayColor];
    button.enabled = NO;
}
```
其中label和button都是这个cell的实例变量，由于cell是自动重用的，所以其他重用此cell的格子也会跟着一起变

# 正确的做法

修改之后，正确的做法应该是：

1、在controller中找到此cell对应的模型

2、修改模型对应的值

3、调用tableView的reloadData方法

4、在dataSource的代理方法里，再调用cell上的设置样式的方法

示意代码：

```
-(void) voteButtonPressed
{
    [myController voteWithCell:self];
}
```

```
-(void) voteWithCell:(CandidateTableViewCell*)cell
{
    RankingView *myView = (RankingView*)self.view;

    // 找到对应的模型
    NSIndexPath *indexPath = [myView.tableView indexPathForCell:cell];
    Candidate *candidate = [candidates objectAtIndex:indexPath.row];

    // 设置新值
    candidate.voteCount++;
    candidate.isVoted = YES;

    // 触发数据加载
    [myView.tableView reloadData];
}
```

```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    CandidateTableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:[CandidateTableViewCell reuseIdentifier] forIndexPath:indexPath];
    Candidate *candidate = [candidates objectAtIndex:indexPath.row];
    [cell setCandidate:candidate isExpired:self.stage != 1 controller:self];
    return cell;
}
```