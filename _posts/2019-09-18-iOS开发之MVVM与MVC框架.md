---
layout: post
title: iOS开发之MVVM与MVC框架
date: 2019-09-18
tag: iOS
---
### MVC框架

1、MVC概述：MVC全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范，注意她是一种框架模式， 而不是设计框架。斯坦福大学有一个教授讲的MVC感觉特别清楚。他用一个图清晰的介绍了MVC框架。展示如下。

<img src="/images/posts/iOS开发之MVVM与MVC框架/iOS开发之MVVM与MVC框架.jpg" > 

他形象用交通双黄线来比喻model、ViewController和view之间的通讯。双黄线表示不能通行。虚线+白色的线可以单方向通讯。
从图中可以形象的看出来model和view之间禁止通讯的。要实现model和view之间的通讯就要通过ViewController来通讯。一般情况下model和ViewController之间的通讯可以通过Notification和KVO广播的方式来实现通讯,而view和ViewController之间的通讯可以用datasource、delegate、Target…action、block的方式来通讯。

2、MVC的优点

MVC是iOS推荐的经典开发框架。model 、ViewController和view之间层级分明。各个模块可重用性好。低耦合性。

3、MVC缺点

view和model之间通讯的很多代码都写在ViewController中，导致ViewController中的代码臃肿。难以定位相应的Bug.

### MVVM框架

1、MVVM框架:MVVM从字面上理解为model（数据模型），view--viewcontroller（视图--视图控制器），viewMode（视图模型），Binder（绑定机制）。这样用ViewModel来实现view和model之间的双向绑定。model绑定view可以用block来通讯。view绑定model可以用KVC或者是RAC等来通讯。这样一来很多逻辑代码就写在来ViewModel中。ViewController只负责他们之间的双向绑定以及必要的逻辑处理代码。这样就减轻了ViewController的负担。ViewController就不再臃肿啦。我也在网上找了两张MVVM的构架图。如下。

<img src="/images/posts/iOS开发之MVVM与MVC框架/iOS开发之MVVM与MVC框架1.png" > 

具体双向绑定和代码的分类下图说得也非常形象

<img src="/images/posts/iOS开发之MVVM与MVC框架/iOS开发之MVVM与MVC框架2.png" > 

2、MVVM优点

减轻了ViewController的负担。ViewController不再臃肿。model和UI的绑定使用block。block可以保存代码块，使得代码的复用性好。结合RAC函数式编程可以大大的加少代码量。
低耦合：view可以独立于model变化和修改，一个viewModel可以绑定到不同的view上。
可重用性：可以把一些视图逻辑放在一个。
独立开发：开发人员可以专注于业务逻辑和数据开发viewModel，设计人员可以专注于页面设计
可测试，可以针对于viewModel来测试.

3、MVVM缺点

bug变得难以调试，当遇到了异常，可能是view的问题，也有可能是model的问题。数据绑定使得bug快速传递到其他地方，要定为原始出问题的地方就变得不那么容易了。
对数据转化需要花费更多的内存。主要来自于对数组内，item又再次包含数组。多次嵌套的类型。需要多次转化才能用来view显示。
对于api返回的数据类型标准化要求较高，提高modelview的复用率，否则容易出现类型爆炸，加大了维护成本。


### MVVM框架Demo讲解

第一：用block实现model和UI的绑定

1、ViewController中用block实现model和UI的绑定。

```
self.mv = [[LCPresentViewModel alloc] init];
    [self.mv initWithBlock:^(NSMutableArray*array){
        __strong typeof (self) strongSelf = weakSelf;
        NSMutableArray * arrays = array;
        [strongSelf.dataArray removeAllObjects];
        [strongSelf.dataArray addObjectsFromArray:arrays];
        [self.myTableView reloadData];
        
    } fail:^(id data){
        
    }];
```

2、ViewModel中模拟下载数据并且刷新的代码如下


```
-(void)loadData{
    dispatch_async(dispatch_get_global_queue(0,0), ^{
        NSArray * array = @[@"转账",@"信用卡",@"充值中心"];
        NSMutableArray *mArray = [NSMutableArray arrayWithArray:array];
        dispatch_async(dispatch_get_main_queue(),^{
            if (self.successBlock) {
                self.successBlock(mArray);
            }
            
        });
    });
}
```
第一：UI和model的绑定

第一种方法：用KVC实现UI和model的绑定.


1添加观察者

```

[self addObserver:self forKeyPath:@"contentkey" options:NSKeyValueObservingOptionNew context:NULL];

```
2、实现观察者方法，当操作UI数据改变时点用

```
-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context{
    NSLog(@"change%@",change);
     NSArray * array = @[@"转账",@"信用卡",@"充值中心"];
    NSMutableArray *mArray = [NSMutableArray arrayWithArray:array];
    [mArray removeObject:change[NSKeyValueChangeNewKey]];
    if (self.successBlock) {
        self.successBlock(mArray);
    }
    
}

```
3、移除观察者

```
-(void)dealloc{
    [self removeObserver:self forKeyPath:@"contentkey"];
}

```

第二种方法：用RAC实现UI和model的绑定.

```
[RACObserve(self, contentkey) subscribeNext:^(id  _Nullable x) {
            NSArray * array = @[@"转账",@"信用卡",@"充值中心"];
            NSMutableArray *mArray = [NSMutableArray arrayWithArray:array];
            [mArray removeObject:x];
            if (self.successBlock) {
                self.successBlock(mArray);
            }
            
        }];

```
第三：点击试图改变数据


```
// 点击cell
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath{
    self.mv.contentkey = self.dataArray[indexPath.row];
    [tableView deselectRowAtIndexPath:indexPath animated:YES];
    
}

```

[iOSMVVM框架的demo链接](https://github.com/iWolfSex/LCMVVMDemo.git)  





<br>
转载请注明：[iWolf的博客](https://iwolfsex.github.io/) » [iOS开发之block](http://iWolf.com/2019/09/iOS开发之MVVM与MVC框架/)  
