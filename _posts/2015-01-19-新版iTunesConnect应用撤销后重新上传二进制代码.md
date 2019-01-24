---
layout: post
title: 新版iTunesConnect应用撤销后重新上传二进制代码
date: 2014-10-19
tag: iOS
---

有时候我们在iTunesConnect提交二进制代码后，应用的状态处于正在等待审核。此时发现应用还存在不完美的地方。显然需要撤销应用。在iTunesConnect里面撤销应用比较简单。问题来了，撤销后怎么重新上传修改后打包的二进制打包文件呢。记得在iTunesConnect改版之前，我们需要先把应用的状态变为黄色的等待上传状态。而在改版后的iTunesConnect里面。下架后的状态始终是红色的已经撤销应用。那么此时怎么办呢？我们我们只需要在Xcode中改变Build的值即可重新打包上传。


<br>
转载请注明：[iWolf的博客](http://iWolf.com) » [新版iTunesConnect应用撤销后重新上传二进制代码](http://iWolf.com/2014/10/新版iTunesConnect应用撤销后重新上传二进制代码/)  


