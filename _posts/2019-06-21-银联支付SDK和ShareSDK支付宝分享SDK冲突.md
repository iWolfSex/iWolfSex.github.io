---
layout: post
title: 银联支付SDK和ShareSDK支付宝分享SDK冲突
date: 2019-06-21
tag: iOS
---

第二次遇到同一个编译错误，必须记录一下。在项目中集成银联的微信支付和支付宝支付功能时编译通不过。错误提示如下。

### Xcode编译错误提示
```
Undefined symbols for architecture arm64:
  "_OBJC_CLASS_$_PayResp", referenced from:
      objc-class-ref in libUMSPosPayOnly.a(UMSPPPayWXPayManager.o)
  "_OBJC_CLASS_$_PayReq", referenced from:
      objc-class-ref in libUMSPosPayOnly.a(UMSPPPayWXPayManager.o)
ld: symbol(s) not found for architecture arm64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
Showing first 200 warnings only

```
记得我们另一个项目在集成支付宝分享功能时也遇到过类似的报错。唯一不同的是另一个项目是先集成的银联的支付功能，在用ShareSDK集成支付宝朋友圈分享功能时报的错。

### 按照错误提示网上给出的解决办法，但并没有解决问题

按照网上搜索到的说法是libUMSPosPayOnly没有支持arm64. 而在我的工程中Valid Architectures和Architectures中均包含了arm64的指令集,这就是说明我需要编译的app最终要支持arm64的,而程序中用到的静态库并没有arm64,所以才导致了出错,因此,需要我们去重新下载一个支持arm64的静态库文件,那么就可以正常编译通过了.
可以通过lipo - info命令查看静态库所支持的架构,打开终端输入查看命令lipo - info xxx.a 

### lipo - info命令查看静态库所支持的架构

```
lipo - info xxx.a 

```

### 解决问题的办法


我记得我们的另一个项目是先做银联的支付功能后才有集成支付宝的分享功能。一开始我也是用ShareSDK的支付宝分享功能来分享支付宝朋友圈。但是集成的过程也是报了莫名其秒的错误。反正看到就头大那种错误。后来我就单独集成了支付宝的支付。没有用ShareSDK分享支付宝朋友圈的功能。才顺利编译通过。以前的经验高数我可能是银联支付SDK和ShareSDK支付宝分享SDK冲突。所以我在这一个项目中先是剔除了ShareSDK分享支付宝朋友圈模块。然后单独集成支付宝分享朋友圈功能。在接着添加银联的微信支付和支付宝支付功能的SDK.结果就顺利编译通过了。我也没有替换libUMSPosPayOnly静态库。也不报上上面的编译错误了。所以应该不是libUMSPosPayOnly没有支持arm64导致的。而是银联支付SDK和ShareSDK支付宝分享SDK冲突。


### Xcode会乱报错误提示吗？

 上面的解决办法证实并不是libUMSPosPayOnly静态库中没有支持arm64. 而在我的工程中Valid Architectures和Architectures中均包含了arm64的指令集导致的。到底是什么原因导致提示Xcode提示上面的错误的。应该是ShareSDK和银联的SDK都同时集成了支付宝的某一资源文件导致的。当时在网上找结局办法时没有找到。
 
 最后想问一下有没有人遇到过这个问题，有不一样的解决办法的？


<br>
转载请注明：[iWolf的博客](https://iwolfsex.github.io/) » [银联支付SDK和ShareSDK支付宝分享SDK冲突](http://iWolf.com/2019/06/银联支付SDK和ShareSDK支付宝分享SDK冲突/)  
