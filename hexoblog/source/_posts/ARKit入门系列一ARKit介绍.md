---
title: ARKit入门系列一ARKit介绍
date: 2017-10-19 14:26:18
tags: [ARKit,iOS11]
---


> 增强现实技术（Augmented Reality，简称 AR），是一种实时地计算摄影机影像的位置及角度并加上相应图像、视频、3D模型的技术，这种技术的目标是在屏幕上把虚拟世界套在现实世界并进行互动。

 AR场景实现所需要的技术和步骤一般包括以下几部分：

* 捕获现实环境的图像：如摄像头
* 三维建模：3D模型的建立
* 传感器追踪：主要追踪现实世界动态物体的六轴变化，这六轴分别是X、Y、Z轴位移及旋转。其中位移三轴决定物体的方位和大小，旋转三轴决定物体显示的区域。
* 坐标识别及转换：3D模型显示在现实图像中不是单纯的frame坐标点，而是一个三维的矩阵坐标。
* 交互：AR还可以与虚拟物体进行交互

# ARKit介绍

ARKit是2017年6月6日，苹果发布iOS11系统所新增框架,它能够帮助我们以最简单快捷的方式实现AR技术功能。

<!-- more -->

ARKit框架提供了两种AR技术，一种是基于3D场景(SceneKit)实现的增强现实，一种是基于2D场景(SpriktKit)实现的增强现实。

要想显示AR效果，必须要依赖于苹果的游戏引擎框架（3D引擎SceneKit，2D引擎SpriktKit），主要原因是游戏引擎才可以加载物体模型。

虽然ARKit框架中视图对象继承于UIView，但是由于目前ARKit框架本身只包含相机追踪，不能直接加载物体模型，所以只能依赖于游戏引擎加载ARKit。

 ARKit到底是什么呢？简单几个字描述清楚移动端的AR平台，高级API。
 
  ![ARkit](/images/arkit/arkit_01_01.jpeg)


  ARKit主要有三层核心技术技术：

  第一层：快速稳定的世界定位 ，包括实时运算，运动定位，无需预设（软硬件）。
  
  ![ARkit](/images/arkit/arkit_01_02.jpeg)

  第二层：平面和边界感知 碰撞测试和光线估算，让虚拟内容和现实环境无缝衔接。

  ![ARkit](/images/arkit/arkit_01_03.jpeg)

  第三层，渲染 支持各种渲染制作工具，目标就是简单易用，和其它插件融合度好。

  ![ARkit](/images/arkit/arkit_01_04.jpeg)


  另外让开发者们惊叫的就是 Unity3D和Unreal也是全线支持。好了，我们看看这个ARKit到底怎么用：苹果AR工程师总结起来也是超级好用，看下图

  ![ARkit](/images/arkit/arkit_01_05.jpeg)

  一切的核心是，首先创建一个ARSession。

  然后就是设置你的ARsession configuration

  可以使用Xcode或者Unity3D的ARKit插件，下面会介绍如何在Xcode里创建ARKit项目


# 环境要求

1.Xcode版本：Xcode9及以上

2.iOS系统: iOS11及以上

3.iOS设备：处理器A9及以上（6S机型及以上）

4.MacOS系统：10.12.4及以上（安装Xcode9对Mac系统版本有要求）
目前只有Bete版本，链接地址:https://developer.apple.com/download/

# ARKit项目创建

 1.打开Xcode9bete版本，新建一个工程，选择 Augmented Reality APP (Xcode9新增),点击next。

 2.包含技术选择SceneKit（3D）,如果2D选择SpriteKit。

 3.此时,Xcode会自动为我们生成一段极其简洁的AR代码。  


# ARKit项目3DDemo代码

```
#import "ViewController.h"

@interface ViewController () <ARSCNViewDelegate>

//ARKit框架中用于3D显示的预览视图
@property (nonatomic, strong) IBOutlet ARSCNView *sceneView;

@end


@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    // Set the view's delegate
    //设置代理
    self.sceneView.delegate = self;

    // Show statistics such as fps and timing information
    //ARKit统计信息
    self.sceneView.showsStatistics = YES;

    // Create a new scene
    //使用模型创建节点（scn格式文件是一个基于3D建模的文件，使用3DMax软件可以创建，这里系统有一个默认的3D飞机）
    SCNScene *scene = [SCNScene sceneNamed:@"art.scnassets/ship.scn"];

    // Set the scene to the view
    //设置ARKit的场景为SceneKit的当前场景（SCNScene是Scenekit中的场景，类似于UIView）
    self.sceneView.scene = scene;
}

- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];

    // Create a session configuration
    //创建一个追踪设备配置（ARWorldTrackingSessionConfiguration主要负责传感器追踪手机的移动和旋转）
    ARWorldTrackingSessionConfiguration *configuration = [ARWorldTrackingSessionConfiguration new];

    // Run the view's session
    // 开始启动ARSession会话（启动AR）
    [self.sceneView.session runWithConfiguration:configuration];
}

- (void)viewWillDisappear:(BOOL)animated {
    [super viewWillDisappear:animated];

    // Pause the view's session
    // 暂停ARSession会话
    [self.sceneView.session pause];
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Release any cached data, images, etc that aren't in use.
} 

@end
```

# ARKit项目2DDemo


```
#import "ViewController.h"
#import "Scene.h"

@interface ViewController () <ARSKViewDelegate>

//ARSKView是ARKit框架中负责展示2D AR的预览视图
@property (nonatomic, strong) IBOutlet ARSKView *sceneView;

@end


@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    // Set the view's delegate
    //设置场景视图代理
    self.sceneView.delegate = self;

    // Show statistics such as fps and node count

    //显示帧率
    self.sceneView.showsFPS = YES;
    //显示界面节点（游戏开发中，一个角色对应一个节点）
    self.sceneView.showsNodeCount = YES;

    // Load the SKScene from 'Scene.sks'
    //加载2D场景（2D是平面的）
    Scene *scene = (Scene *)[SKScene nodeWithFileNamed:@"Scene"];

    // Present the scene
    //AR预览视图展现场景（这一点与3D视图加载有区别）
    [self.sceneView presentScene:scene];
}

- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];

    // Create a session configuration
    //创建设备追踪设置
    ARWorldTrackingSessionConfiguration *configuration = [ARWorldTrackingSessionConfiguration new];

    // Run the view's session

    //开始启动AR
    [self.sceneView.session runWithConfiguration:configuration];
}

- (void)viewWillDisappear:(BOOL)animated {
    [super viewWillDisappear:animated];

    // Pause the view's session
    [self.sceneView.session pause];
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Release any cached data, images, etc that aren't in use.
}

#pragma mark - ARSKViewDelegate


//点击界面会调用，类似于touch begin方法  anchor是2D坐标的瞄点
- (SKNode *)view:(ARSKView *)view nodeForAnchor:(ARAnchor *)anchor {
    // Create and configure a node for the anchor added to the view's session.

    //创建节点（节点可以理解为AR将要展示的2D图像）
    SKLabelNode *labelNode = [SKLabelNode labelNodeWithText:@"👾"];
    labelNode.horizontalAlignmentMode = SKLabelHorizontalAlignmentModeCenter;
    labelNode.verticalAlignmentMode = SKLabelVerticalAlignmentModeCenter;
    return labelNode;
}

- (void)session:(ARSession *)session didFailWithError:(NSError *)error {
    // Present an error message to the user
    
}

- (void)sessionWasInterrupted:(ARSession *)session {
    // Inform the user that the session has been interrupted, for example, by presenting an overlay
    
}

- (void)sessionInterruptionEnded:(ARSession *)session {
    // Reset tracking and/or remove existing anchors if consistent tracking is required
    
}

@end
 ```

 参考文章：
1. [http://blog.csdn.net/u013263917/article/details/72903174](http://blog.csdn.net/u013263917/article/details/72903174)

2. [http://www.jianshu.com/p/24a8f418c9aa?from=timeline](http://www.jianshu.com/p/24a8f418c9aa?from=timeline)