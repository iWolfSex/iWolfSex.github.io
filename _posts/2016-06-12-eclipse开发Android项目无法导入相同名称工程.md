---
layout: post
title: eclipse开发Android项目无法导入相同名称工程
date: 2016-06-12
tag: iOS
---

作为ios开发程序员，要接手公司的安卓项目，对于安卓开发还很陌生。在这台路上肯定会遇到各种坑。本来是常识问题就会折腾半天。我就遇到了一个很无奈的问题，我本想是想在eclipse中删除项目，就直接在eclipse的package Explorer下删除该项目。我想重新从工作空间里面导入该项目结果，不管如何都添加不上。提示有相同名称的项目已经存在。于是各种百度google最后发现是删除项目的方法有错。只是删除eclipse的package Explorer下删除该项目还不行。还得删除workspace下物理删除那个同名的工程 文件就可以了。你也可以先把workspace路径下的项目copy到其他文件，在删除。然后重新导入的时候勾选copy projects inito workspace。
 
<br>
转载请注明：[iWolf的博客](http://iWolf.com) » [eclipse开发Android项目无法导入相同名称工程](http://iWolf.com/2016/06/eclipse开发Android项目无法导入相同名称工程/)  


