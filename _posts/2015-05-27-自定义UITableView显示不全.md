---
layout: post
title: 自定义UITableView显示不全
date: 2015-05-27
tag: iOS
---

我在开发过程中，遇到了自定义UITableView显示不全的情况，有两行cell始终拉不到底部。估计是我自定义cell时改变了cell的高导致的。我的解决办法是让UITableView多显示两行。下面的数组_allAlarmArray是数据源。在UITableView的协议函数中返回几行的函数。
```
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
return [_allAlarmArray count]+2;
}
```
一开始我以为就这样就可以了。但是运行后的效果没有任何变化。这个需要注意了，此处还需要在初始化cell的那个协议函数里面对多出的两行做处理。就是让多出的两行什么也数据也不显示。即在协议函数- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath中添加几句代码如下。
```
if (indexPath.row >=  [_allAlarmArray count]) {
cell.imageView.image=[UIImage imageNamed:@""];
cell.textLabel.text = @"";
cell.detailTextLabel.text = @"";
return cell;
}。
``` 
<br>
转载请注明：[iWolf的博客](http://iWolf.com) » [自定义UITableView显示不全](http://iWolf.com/2015/05/自定义UITableView显示不全/)  

