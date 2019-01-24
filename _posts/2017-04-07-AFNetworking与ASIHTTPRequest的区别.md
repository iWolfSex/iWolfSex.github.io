---
layout: post
title: AFNetworking与ASIHTTPRequest的区别
date: 2017-04-07
tag: iOS
---

### 一、底层实现

>* 1、AFN的底层实现基于OC的NSURLConnection和NSURLSession
>* 2、ASI的底层实现基于纯C语言的CFNetwork框架
>* 3、因为NSURLConnection和NSURLSession是在CFNetwork之上的一层封装，因此ASI的运行性能高于AFN
AFNetworking地址: https://github.com/AFNetworking/AFNetworking

###  二、对服务器返回的数据处理

>* 1、ASI没有直接提供对服务器数据处理的方式，直接返回的是NSData/NSString
>* 2、AFN提供了多种对服务器数据处理的方式

```
(1)JSON处理-直接返回NSDictionary或者NSArray

(2)XML处理-返回的是xml类型数据，需对其进行解析

(3)其他类型数据处理
```
###  三、监听请求过程

>* 1、AFN提供了success和failure两个block来监听请求的过程（只能监听成功和失败）
* success : 请求成功后调用
* failure : 请求失败后调用
>* 2、ASI提供了3套方案，每一套方案都能监听请求的完整过程

```
（监听请求开始、接收到响应头信息、接受到具体数据、接受完毕、请求失败）
* 成为代理，遵守协议，实现协议中的代理方法
* 成为代理，不遵守协议，自定义代理方法
* 设置block
。。。
```

### 四、在文件下载和文件上传的使用难易度

>* 1、AFN
```
不容易实现监听下载进度和上传进度 】 不容易实现断点续传 、
```
*一般只用来下载不大的文件

>* 1、ASI
```
非常容易实现下载和上传 非常容易监听下载进度和上传进度
非常容易实现断点续传 下载大文件或小文件均可
```
>* 2、实现下载上传推荐使用ASI

### 五、网络监控

>* 1、AFN自己封装了网络监控类，易使用
>* 2、ASI使用的是Reachability，因为使用CocoaPods下载ASI时，会同步下载Reachability，但Reachability作为网络监控使用较为复杂（相对于AFN的网络监控类来说）
>* 3、推荐使用AFN做网络监控-AFNetworkReachabilityManager
六、ASI提供的其他实用功能

>* 1、控制信号旁边的圈圈要不要在请求过程中转
>* 2、可以轻松地设置请求之间的依赖：每一个请求都是一个NSOperation对象
>* 3、可以统一管理所有请求（还专门提供了一个叫做ASINetworkQueue来管理所有的请求对象）
```
* 暂停/恢复/取消所有的请求
* 监听整个队列中所有请求的下载进度和上传进度
```
<br>
转载请注明：[iWolf的博客](http://iWolf.com) » [AFNetworking与ASIHTTPRequest的区别](http://iWolf.com/2017/04/AFNetworking与ASIHTTPRequest的区别/)  


