---
layout: post
title: IIViewDeckController拖动事件与百度地图拖动事件冲突
date: 2015-07-10
tag: iOS
---

在使用第三方（IIViewDeckController）的侧边栏和百度地图同时使用时。发现百度地图的拖动事件被侧边栏截取了，导致拖动地图时地图移动缓慢，几乎无法滑动。在网上苦苦搜寻了解决方案，最终搜索到比较靠谱的http://www.cocoachina.com/bbs/read.php?tid-252890-page-2.html链接里面11楼的评论。但是我按照他的办法添加代码，在我这边还是没有效果。于是我就在IIViewDeckController第三类里面去找- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch，这个函数。最终在这个协议函数里面添加了几行代码，大概意思就是如果我拖动的是百度地图页面。我就返回NO。如果不是百度地图页面我就返回YES。最终完美解决了百度地图不能华东的bug。如下是源码。


```
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch {  
    if (self.panningGestureDelegate && [self.panningGestureDelegate respondsToSelector:@selector(gestureRecognizer:shouldReceiveTouch:)]) {  
        BOOL result = [self.panningGestureDelegate gestureRecognizer:gestureRecognizer  
                                                  shouldReceiveTouch:touch];  
        if (!result) return result;  
    }  
  
    if ([[touch view] isKindOfClass:[UISlider class]])  
        return NO;  
  
    _panOrigin = self.slidingControllerView.frame.origin;  
    BOOL isMap = NO;  
    for (UIView* theview in [touch.view subviews] ) {  
        if ([theview isKindOfClass:[BMKPinAnnotationView class]]) {  
            isMap=YES;  
        }  
    }  
    if (isMap) {  
        return NO;  
    }else{  
        return YES;  
    }  
}

```
 
<br>
转载请注明：[iWolf的博客](http://iWolf.com) » [IIViewDeckController拖动事件与百度地图拖动事件冲突](http://iWolf.com/2015/07/IIViewDeckController拖动事件与百度地图拖动事件冲突/)  


