---
layout: post
title: CocoadrawRect
date: 2014-03-22
tag: iOS
---

前一段时间学习IOS和Mac OS X绘图，在这里分别对IOS和Mac OS X下的绘图举一些列。

一、在IOS画一个矩形：

- (void)drawRect:(CGRect)rect{


CGContextRef contextRuler=UIGraphicsGetCurrentContext();//获取画布

CGContextSetFillColorWithColor(contextRuler, [[UIColorwhiteColor]CGColor]);设置颜色

CGContextFillRect(contextRuler,CGRectMake((self.frame.size.width/2)-RADIUS/2,self.frame.size.height*1/10,RADIUS , self.frame.size.height*6/8-self.frame.size.height*1/10));设置绘画路径

CGContextDrawPath(contextPointer,kCGPathFill);//绘制路径

}

二、在OS X画一个矩形

- (void)drawRect:(NSRect)dirtyRect{


[superdrawRect:dirtyRect];

    CGContextRef myContext=[[NSGraphicsContextcurrentContext]graphicsPort];//获取画布

    CGContextSetFillColorWithColor(myContext, [[NSColorgrayColor]CGColor]);设置

    CGContextFillRect(myContext,CGRectMake(0,0,self.frame.size.width,self.frame.size.height));绘制区域

}

仔细看了一下，就是获取画布有些不一样。

三、还有一个重要的知识点就是在重绘。我的理解其是相当于调用- (void)drawRect:(NSRect)dirtyRect；

1、你需要重绘的时候IOS应该调这个函数[self setNeedsDisplay];

2、如果在Mac OS X也需要重绘的时候应该调[self setNeedsDisplay:YES];
 
<br>
转载请注明：[iWolf的博客](http://iWolf.com) » [Cocoa drawRect](http://iWolf.com/2014/03/CocoadrawRect/)  


