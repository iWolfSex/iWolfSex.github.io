---
layout: post
title: Json转换成NSDictionary
date: 2015-05-26
tag: iOS
---

NSstring*json=@"\"s\":{\"n\":\"刹车\"}";

NSData* alarmInfoData = [json dataUsingEncoding:NSUTF8StringEncoding];

NSDictionary * alarmInfoDic =[NSJSONSerialization JSONObjectWithData:alarmInfoData options:NSJSONReadingMutableLeaves error:nil];
 
<br>
转载请注明：[iWolf的博客](http://iWolf.com) » [Json转换成NSDictionary](http://iWolf.com/2015/05/Json转换成NSDictionary/)  


