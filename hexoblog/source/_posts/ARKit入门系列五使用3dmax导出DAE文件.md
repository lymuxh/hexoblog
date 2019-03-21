---
title: ARKit入门系列五使用3dmax导出DAE文件
date: 2017-11-06 18:21:55
tags: [ AR , 3dmax , DAE]
---

## DAE文件

 DAE文件格式是3D交互文件格式，一般用于多个图形程序之间交换数字数据，Autodesk专有并在COLLADA（COLLAborative Design Activity）基础上改进创建的XML框架的文件格式。COLLADA文件格式是由SONY改进并有SONY和Khronos共同开发的。

　　DAE是一种3D模型，可被flash 导入。3Dmax与maya需要安装dae输出插件才可以打开，输出成后缀为dae的文件。谷歌地球的模型就是后缀为dae的文件。

　　 Autodesk的Collada，是Autodesk自家产的，你要用必然要买正版的Autodesk产品；OpenCollada是一个具有开放标准的Collada，有很多免费的3D软件支持。 

　　 用 Autodesk的Collada的导出的DAE文件，只会导出模型， 并不会导出模型的贴图。这时候我们就需要安装第三方插件OpenCollada。 

小伙伴们下载OpenCollada，只需将COLLADAMax.dle放到3DMAX的安装目录中的 plugins 文件夹中就行了，然后重启3DMAX就OK了。

<!-- more -->

   ![ARkit](/images/arkit／arkit_05_01.jpg)

然后我们打开一个后缀为max的3D模型，然后导出，在保存类型中就多了OpenCollada这一项，如下图：

   ![ARKit与SceneKit](/images/arkit／arkit_05_02.jpg)

然后选择OpenCollada这一项，点击保存，会出现如下界面：

   ![ARKit与SceneKit](/images/arkit／arkit_05_03.jpg)

勾选中”copy Images“ ，点击OK。

然后导出后会有一个images文件夹，里面存放的就是贴图：

   ![ARKit与SceneKit](/images/arkit／arkit_05_04.jpg)

只有保存dae模型与images文件夹在同一目录，在xcode 中打开dae模型就会按路径找到贴图，并且自动贴了上去，不用任何手动贴图，特别方便。

参考文章：

[http://blog.csdn.net/pzhtpf/article/details/50445223](http://blog.csdn.net/pzhtpf/article/details/50445223) 