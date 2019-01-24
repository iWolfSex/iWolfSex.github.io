---
layout: post
title: 使用containerView在UIViewController实现staticCell的使用
date: 2017-05-19
tag: iOS
---

今天在做项目在UIViewController中的UItableview使用staticCell，刚在storyboard中的tableView设置完staticCell，就出现 Static table views are only valid when embedded in UITableViewController instances警告。是说staticCell只能在UItableviewController使用。那么在UIViewController中还可以使用staticCell吗？答案是肯定的。这要借助container来实现。container就是一个容器，可以放ViewController。我们在storyboard拖一个container在UIViewController中，然后在拖一个UItableviewController在storyboard中。然后从containerView拖一segue选择embed。这样就能实现在UIViewController实现staticCell的使用。在如下图

<img src="/images/posts/堆与栈/堆与栈1.jpg" > 

<img src="/images/posts/堆与栈/堆与栈2.jpg" > 

<br>
转载请注明：[iWolf的博客](http://iWolf.com) » [堆与栈](http://iWolf.com/2017/05/使用containerView在UIViewController实现staticCell的使用/)  


