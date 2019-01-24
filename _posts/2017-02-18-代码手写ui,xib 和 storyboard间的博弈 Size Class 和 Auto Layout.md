---
layout: post
title: 代码手写ui,xib 和 storyboard间的博弈 Size Class 和 Auto Layout
date: 2017-02-18
tag: iOS
---

1、各种布局的简介。

2、各种布局的利弊。

3、如何取舍各种布局。

4、Size Class 和Auto Layout 的使用。

5、各种布局适配iOS横竖屏旋转。

一、各种布局的简介。

    随着iOS发展到今天，UI上的布局逐渐形成了三大主流派：

    第一种：使用纯代码写UI及其布局。比如我们的CMSMoblie项目。就是典型的例子。

    第二种：使用单个xib文件组织viewController和view。比如我们的运维宝项目。

    第三种：使用storyBoard来通过单个或很少的几个文件来构建全部UI。比如我们的CeiBar2项目。

    其实这三种方式布局各有优劣。所以各有适合的应用场景。

二 、各种布局的利弊

第一种：代码手写UI

优点：1、代码可写UI可从用性高。

2、适合大型项目多人开发。

缺点：1、写代码速度慢。

2、不好适配横竖屏旋转。

第二种：xibs

优点：1、可以少写大量代码，从而提高开发速度。

     2、xib的设计更加完美的体现了MVC的设计模式。

缺点：1、xib文件内容过于复杂，可读性很差。

 2、导致多人开发提交合并代码的痛苦。

 3、xib没有逻辑判断，也很难在运行是进行配置，不像使用代码那样无所不能。

第三种：storyBoard

优点：1、在StoryBoard中不仅可以看到每个ViewController的布局样式，也可以明确地知道各个ViewController之间的转换关系。

2、相对于单个的xib，其代码需求更少，也由于集合了各个xib，使得对于界面的理解和修改的速度也得到了更大提升。

缺点：1、StoryBoard面临的最大问题就是多人协作。

2、StoryBoard的另外的挑战来源于ViewController的重用和自定义的view的处理。

3、因为相对于单个xib来说，StoryBoard文件往往更大，加载速度也相应变慢。

三、如何取舍各种布局。

    大型项目、自定义控件多、多人开发、需要多个项目重用的、建议用纯代码编写UI界面。

    小项目、追求开发速度的项目、需要支持横竖屏旋转的项目。可以用xib和StoryBoard。

四、Size Class 和Auto Layout 的使用。

1、Size Class简介

Size Classes。在 iOS8 中，我们不用再像以前那样，一个页面新建多个 xib 文件来适配不同类型的屏幕，现在我们可以把各种尺寸屏幕的适配工作放在一个文件中完成，然后可以通过不同类别的 Size 来定制各种尺寸的界面。换句话说，你眼前的 Storyboard 不是一个普通的 Storyboard ，而是一个九合一的 Storyboard ，可以管理九种类型的屏幕。

 

对于宽度和高度而言，都有三种情况：紧凑 (Compact) 、任意 (Any) 、正常 (Regular) ，所以一共有9个类别，在设置 Size Class 的时候页面会有提示。比如宽为 Compact 高为 Any 的情况，提示为 3.5-inch、4-inch、4.7-inch的横竖状态下的屏幕：

2、AutoLayout和Autoresizing Mask的区别

描述：每个view的size inspector中都有一个红色线条的Autoresizing的指示器和相应的动画缩放的示意图，这就是Autoresizing Mask。在iOS6之前，关于屏幕旋转的适配和iPhone，iPad屏幕的自动适配，基本都是由Autoresizing Mask来完成的。但是随着大家对iOS app的要求越来越高，以及已经以及今后可能出现的多种屏幕和分辨率的设备来说，Autoresizing Mask显得有些落伍和迟钝了

第一点：AutoLayout可以指定任意两个view的相对位置，而不需要像Autoresizing Mask那样需要两个view在直系的view hierarchy中。

第二点：AutoLayout不必须指定相等关系的约束，它可以指定非相等约束（大于或者小于等）；而Autoresizing Mask所能做的布局只能是相等条件的。

第三点：AutoLayout可以指定约束的优先级，计算frame时将优先按照满足优先级高的条件进行计算。

3、Auto Layout 的使用。

简介：Auto Layout是在WWDC2012上被引入到iOS中的，从iOS6.0以后就开始支持，但是大多数的开发者还是习惯使用传统的UI布局方式，虽然有一大部分开发者早已使用了Auto Layout，这其中大多数的开发者是在拖拽IB文件或者是使用StoryBoard时才会选择用Auto Layout的布局方式。

Auto Layout是一种基于约束的、描述性的布局系统。也就是使用约束条件来描述布局，View的Frame会根据这些描述来进行计算。

在iOS6.0以后加入了一个新类：NSLayoutConstraint。我们可以使用可视化格式化语言Visual Format Languag的方式创建约束。

五、各种布局适配iOS横竖屏旋转。

1、结合我们的实际项目来讲解。

本文参考文档：

http://www.itjhwd.com/adaptive-layout-for-iphone6-1/

http://www.cocoachina.com/swift/20141013/9893.html

http://www.2cto.com/kf/201409/334180.html

http://blog.csdn.net/dongbaojun_ios/article/details/12566529

 
<br>
转载请注明：[iWolf的博客](http://iWolf.com) » [代码手写ui,xib和storyboard间的博弈SizeClass和AutoLayout](http://iWolf.com/2017/02/代码手写ui,xib和storyboard间的博弈SizeClass和AutoLayout/)  


