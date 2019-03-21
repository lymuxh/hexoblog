---
title: ARKit入门系列二工作原理和流程
date: 2017-10-20 10:36:54
tags: [iOS11,ARKit]
---


# SceneKit

 苹果自家的3D 引擎 `SceneKit`,是一个OC 框架，开始之前，先熟悉一下SceneKit 的三维坐标系：

 ![ARKit与SceneKit](/images/arkit/arkit_02_01.jpeg)

  很清楚的看到，SceneKit 中的坐标系是右手坐标系(笛卡尔坐标系)，如果需要与其他3D框架共享数据，先了解其框架是右手坐标系还是左手坐标系。其实也很好转化，就是Z 轴的正负不一样而已。

  在开始开发之前，一定要了解下面这几个非常重要的类
|    类 / 协议         |  描述      |    备注            |
|-------------|----------------|-------------------|
| SCNView & SCNSceneRenderer | 类似UIView，用来显示 SceneKit 的内容，定义了一些代理方法，可以用 addSubView 方法添加到UiView 中. | |
| SCNScene | SceneKit 内容的容器. 你可以从3D建模工具生成的.dae文件中加载一个场景，或者用代码创建一个 ，然后把它显示在视图上. | |
| SCNNode | 一个场景的基本构建块，你可以把摄像机，灯光，几何体附加到节点上 | |
| SCNGeometry | 一个可以连接到一个节点的三维物体。一个几何体（有时称为模型或网格）只定义了一个可见物体的形状。要定义对象的表面颜色图案，你必需要给几何体要附加材料。然后给材料贴图，或者上色，这个几何体表面才会有颜色，或者图案。你可以从3D建模工具生成的.dae文件中加载一个几何体，也可以用代码创建，SceneKit 提供了几种常见几何体，是SCNGeometry 的子类，比如长方体，球，圆柱球等等，后面我们会写一个demo会把官方提供的几何体给大家列出来，给大家一个直观的感受。 当然我们也可以用三维坐标，法向量自定义几何体，也可以讲一个2D 图案转化成一个具有深度(厚度)的三维几何体。后面应该专门有一篇会讲到利用贝塞尔曲线将一个2D 图案转化成一个具有深度(厚度)的三维几何体。| |
| CNMaterial | 材质，由于在3D建模工具中呈现球形，所以也叫材质球。上色，贴图全靠它。| |
| SCNLight |光源可以附加到节点上，在渲染场景中提供着色. | |
| SCNCamera | 虚拟摄像机可以附加到节点上，提供了一个场景的视图。| |

<!-- more -->

# ARKit 与 SceneKit关系

  初次接触 ARKit ,很多人会为其复杂的架构关系而感到畏惧。这里简介的方式介绍一下苹果原生AR（虚拟增强现实）， ARKit并不是一个独立就能够运行的框架，而是必须要SceneKit一起用才可以，换一句话说，如果只有`ArKit` ,而没有`SceneKit` ，那么ARKit和一般的相机没有任何区别。两者配合实现AR功能。 

  ARkit主要实现相机捕捉显示世界图像，SceneKit主要负责显示虚拟3D模型。

  ![ARKit与SceneKit](/images/arkit/arkit_scencekit.png)

 1. `<ARKit>`框架中中显示3D虚拟增强现实的视图`ARSCNView`继承于`<SceneKit>`框架中的`SCNView`,而`SCNView`又继承于`<UIKit>`框架中的`UIView`。

 * `UIView`的作用是将视图显示在iOS设备的window中，`SCNView`的作用是显示一个3D场景，`ARScnView`的作用也是显示一个3D场景，只不过这个3D场景是由摄像头捕捉到的现实世界图像构成的。

 2. `ARSCNView`只是一个视图容器，它的作用是管理一个`ARSession`,笔者称之为AR会话。


 3. 在一个完整的虚拟增强现实体验中，`<ARKit>`框架只负责将真实世界画面转变为一个3D场景，这一个转变的过程主要分为两个环节：由`ARCamera`负责捕捉摄像头画面，由`ARSession`负责搭建3D场景。

 4. 在一个完整的虚拟增强现实体验中，将虚拟物体现实在3D场景中是由`<SceneKit>`框架来完成中：每一个虚拟的物体都是一个节点`SCNNode`,每一个节点构成了一个场景`SCNScene`,无数个场景构成了3D世界。

 综上所述，`ARKit`捕捉3D现实世界使用的是自身的功能，这个功能是在iOS11新增的。而`ARKit`在3D现实场景中添加虚拟物体使用的是父类`SCNView`的功能，这个功能早在iOS8时就已经添加（`SceneKit`是iOS8新增），可以简单的理解为：`ARSCNView`所有跟场景和虚拟物体相关的属性及方法都是自己父类SCNView的。

# ARSCNView 与 ARSession

 1. `ARKit`提供两种虚拟增强现实视图，他们分别是3D效果的`ARSCNView`和2D效果的`ARSKView`，无论是使用哪一个视图都是用了相机图像作为背景视图（这里可以参考iOS自定义相机中的预览图层），而这一个相机的图像就是由`<ARKit>`框架中的相机类`ARCamera`来捕捉的。

 2. `ARSCNView`与`ARCamera`两者之间并没有直接的关系，它们之间是通过AR会话，也就是`ARKit`框架中非常重量级的一个类`ARSession`来搭建沟通桥梁的。

    在iOS框架中，凡是带`session`或者`context`后缀的，这种类一般自己不干活，作用一般都是两个：

      1.管理其他类，帮助他们搭建沟通桥梁，好处就是解耦

      2.负责帮助我们管理复杂环境下的内存

      `context`与`session`不同之处是：一般与硬件打交道，例如摄像头捕捉`ARSession`，网卡的调用`NSURLSession`等使用的都是`session`后缀。没有硬件参与，一般用`context`，如绘图上下文，自定义转场上下文等。

 3. 要想运行一个`ARSession`会话，你必须要指定一个称之为会话追踪配置的对象:`ARSessionConfiguration`,`ARSessionConfiguration`的主要目的就是负责追踪相机在3D世界中的位置以及一些特征场景的捕捉（例如平面捕捉），这个类本身比较简单却作用巨大。

 `ARSessionConfiguration`是一个父类，为了更好的看到增强现实的效果，苹果官方建议我们使用它的子类。`ARWorldTrackingSessionConfiguration`，该类只支持A9芯片之后的机型，也就是iPhone6s之后的机型。

# ARWorldTrackingSessionConfiguration 与 ARFrame

 1. `ARSession`搭建沟通桥梁的参与者主要有两个`ARWorldTrackingSessionConfiguration`与`ARFrame`

 2. `ARWorldTrackingSessionConfiguration`（会话追踪配置）的作用是跟踪设备的方向和位置,以及检测设备摄像头看到的现实世界的表面。它的内部实现了一系列非常庞大的算法计算以及调用了你的iPhone必要的传感器来检测手机的移动及旋转甚至是翻滚,我们无需关心内部实现，`ARKit`框架帮助我们封装的非常完美，只需调用一两个属性即可。

 3. 当`ARWorldTrackingSessionConfiguration`计算出相机在3D世界中的位置时，它本身并不持有这个位置数据，而是将其计算出的位置数据交给`ARSession`去管理（与前面说的`session`管理内存相呼应），而相机的位置数据对应的类就是`ARFrame`。`ARSession`类一个属性叫做`currentFrame`，维护的就是`ARFrame`这个对象。

 4. `ARCamera`只负责捕捉图像，不参与数据的处理。它属于3D场景中的一个环节，每一个3D Scene都会有一个Camera，它决定了我们看物体的视野。



# AR工作流程

ARKit框架工作流程可以参考下图:

![ARKit流程](/images/arkit/arkit_workflow.png)

1. `ARSCNView`加载场景`SCNScene`

2. `SCNScene`启动相机`ARCamera`开始捕捉场景

3. 捕捉场景后`ARSCNView`开始将场景数据交给`Session`

4. `Session`通过管理`ARSessionConfiguration`实现场景的追踪并且返回一个`ARFrame`

5. 给`ARSCNView`的`scene`添加一个子节点（3D物体模型）

`ARSessionConfiguration`捕捉相机3D位置的意义就在于能够在添加3D物体模型的时候计算出3D物体模型相对于相机的真实的矩阵位置。

在3D坐标系统中，有一个世界坐标系和一个本地坐标系。类似于`UIView`的`Frame`和`Bounds`的区别，这种坐标之间的转换可以说是ARKit中最难的部分。


参考文章：
1. [http://blog.csdn.net/u013263917/article/details/73038519](http://blog.csdn.net/u013263917/article/details/73038519)

2. [http://www.chinaar.com/ARKit/5212.html](http://www.chinaar.com/ARKit/5212.html)