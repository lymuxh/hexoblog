---
title: ARKit入门系列四SceneKit简介
date: 2017-11-06 17:36:40
tags: [ARKit , SceneKit]
---


ARSCNView 继承自SceneKit的 SCNView，SceneKit展示的内容在ARSCNView上也可以展示，SceneKit作为构建3D场景的框架，且可以与Core Animation和SpriteKit无缝交互。在SceneKit中可以直接引入COLLADA行业标准文件制作好的3D模型或场景。

与SpriteKit一样，SceneKit通过场景（SCNScene）来显示物体，场景包涵在SCNView。场景内同样是以节点的结构来呈现物体。场景里可以包含这些类型的项目：

* 几何体 代码建立的3D对象或从文件加载的3D模型，在它们上可以附加不同的材料来达成控制颜色，纹理，反光等效果。

* 照相机 这是你观察这个场景的入口，可以设置位置和视角。

* 光源 有各种光源后面会提到。

* 物理实体 受物理特效控制的各种有实体的物体，也有许多种。

使用SceneKit的方法不止有在SCNView内部来使用。还可以在Core Animation层的层次结构中使用SCNLayer。使用SCNRender来渲染你自己的OpenGL渲染器。

## 添加场景

首先要做的当然是添加场景。

```
//获取SCNView
let scnView = self.view as! SCNView
//我自己的场景
let myScenes = myScene()
//将SCNView的场景设置为我的场景
scnView.scene = myScenes

```
<!-- more -->

## 添加照相机

照相机是你观察你构建的3D场景的眼睛。

```
//创建视角，可以说是整个场景的入口，没有视角没法观察一个3D场景
let myCamera = SCNCamera()
myCamera.xFov = 45
myCamera.yFov = 45
let myCameraNode = SCNNode()
myCameraNode.camera = myCamera
myCameraNode.position = SCNVector3(0, 0, 20)
//向场景添加节点与SpriteKit有些不一样
myScenes.rootNode.addChildNode(myCameraNode)

```

## 添加3D对象

```
//创建胶囊
let capsule = SCNCapsule(capRadius: 2.5, height: 10)
//SCNCapsule是SCNGeomery的一个子类，通过这个类可以创建更多的形状
let capsuleNode = SCNNode(geometry: capsule)
capsuleNode.position = SCNVector3(-15, -2.8, 0)//节点的默认位置是0，0，0
capsuleNode.name = "myCapsule"
myScenes.rootNode.addChildNode(capsuleNode)

```

## 添加光源

光源有许多种：

* 环境光源，它在整个场景内投射均匀光
* 泛光源，这是用的最多的，就是点光源，向各个方向投射光
* 平行光源，在单个方向投射光
* 聚光源，在给定方向从单个位置投射光

```
//添加环境光源
let ambientLight = SCNLight()
//光源的类型，这里是环境光源
ambientLight.type = SCNLightTypeAmbient
ambientLight.color = UIColor(white: 0.25, alpha: 1.0)
let myAmbientLightNode = SCNNode()
myAmbientLightNode.light = ambientLight
myScenes.rootNode.addChildNode(myAmbientLightNode)

//添加泛光源
let omniLight = SCNLight()
omniLight.type = SCNLightTypeOmni
omniLight.color = UIColor(red: 0, green: 0, blue: 1, alpha: 1)
let omniLightNode = SCNNode()
omniLightNode.light = omniLight
omniLightNode.position = SCNVector3(-10, 8, 5)
myScenes.rootNode.addChildNode(omniLightNode)
添加动画

//向胶囊节点添加动画
let moveUpDownAnimation = CABasicAnimation(keyPath: "position")//这里的这个keyPath很重要，不是随便写一个就好的
moveUpDownAnimation.byValue = NSValue(SCNVector3: SCNVector3(30, 0, 0))//移动的坐标
moveUpDownAnimation.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseInEaseOut)//移动速度的曲线
moveUpDownAnimation.autoreverses = true//是否自动返回
moveUpDownAnimation.repeatCount = Float.infinity//重复次数
moveUpDownAnimation.duration = 10.0//持续时间
capsuleNode.addAnimation(moveUpDownAnimation, forKey: "updown")//这里的这个keyPath貌似随便写就可以

//胶囊节点的子节点，一个文本
let text = SCNText(string: "BaLaLaLa", extrusionDepth: 1)
text.font = UIFont.systemFontOfSize(2)
let textNode = SCNNode(geometry: text)
textNode.position = SCNVector3(-5, 6, 0)
capsuleNode.addChildNode(textNode)

//在一个节点上添加多个动画结果会复合，即便是继承自父节点的动画也同样一起复合

let rotate = CABasicAnimation(keyPath: "eulerAngles")
rotate.byValue = NSValue(SCNVector3: SCNVector3(Float(0), Float(M_PI * 2), Float(0)))
rotate.repeatCount = Float.infinity
rotate.duration = 5.0
textNode.addAnimation(rotate, forKey: "rotation")
```

## 使用材料

材料可以用来改变物体表面的样子，材料使用SCNMaterial类来表示，材料对象拥有许多许多属性，比如：

* diffuse（材料的基本颜色，纹理等）
* specular（材料的亮度以及该如何反射光）
* emissive（材料发光时的样子）
* normal（又称为法向，设置材料表面更多的细节）

每个属性都拥有contents属性，给这个属性设置不同的内容来设置这些属性的样子：

* 颜色
* 图像（NSImage等）
* SpriteKit场景
* SpriteKit纹理

```
//使用材料
let redMetallicMateril = SCNMaterial()
//contents设置为颜色
redMetallicMateril.diffuse.contents = UIColor.blueColor()
redMetallicMateril.specular.contents = UIColor.whiteColor()
redMetallicMateril.shininess = 1.0
//一个物体的材料是不唯一的，故传进去一个数组
capsule.materials = [redMetallicMateril]

let noiseTexture = SKTexture(noiseWithSmoothness: 0.25, size: CGSize(width: 512, height: 512), grayscale: true)
let noiseMaterial = SCNMaterial()
//contents设置为SpriteKit纹理
noiseMaterial.diffuse.contents = noiseTexture
text.materials = [noiseMaterial]

//法线贴图
let noiseNormalMapTexture = noiseTexture.textureByGeneratingNormalMapWithSmoothness(1, contrast: 1.0)
redMetallicMateril.normal.contents = noiseNormalMapTexture
```

## 命中检测

这个说白了就是监测你点击了哪个物体，首先当然要添加手势。

```
//命中检测
let tapRecognizer = UITapGestureRecognizer(target: self, action: "tapped:")
let longTapRecoginizer = UILongPressGestureRecognizer(target: self, action: "longTapped:")
scnView.addGestureRecognizer(tapRecognizer)
scnView.addGestureRecognizer(longTapRecoginizer)
scnView.userInteractionEnabled = true
```

接下来添加手势对应的执行函数

```
func tapped(tapRecognize: UIGestureRecognizer){
    if tapRecognize.state == UIGestureRecognizerState.Ended {
     let scnView = self.view as! SCNView
    //检测点击到哪个，返回被点到的物体，这里返回的数组里包含了你点击的点顺着屏幕法线穿过的所有的物体
    let hits = scnView.hitTest(tapRecognize.locationInView(tapRecognize.view), options: nil) as [SCNHitTestResult]
    for hit in hits {
    if let theMaterial = hit.node.geometry?.materials[0] {
    let hightLightAnimation = CABasicAnimation(keyPath: "contents")
    hightLightAnimation.fromValue = UIColor.blackColor()
     hightLightAnimation.toValue = UIColor.yellowColor()
    hightLightAnimation.autoreverses = true
    hightLightAnimation.repeatCount = 2
    hightLightAnimation.duration = 1
    theMaterial.emission.addAnimation(hightLightAnimation, forKey: "heightLight")
            }
        }
        }
    }
```

## 为节点添加约束

```
//添加指向胶囊节点这个约束
let lookAtConstraint = SCNLookAtConstraint(target: capsuleNode)
//使其只围绕一个轴转动
lookAtConstraint.gimbalLockEnabled = true
pointerNode.constraints = [lookAtConstraint]
```


## 从COLLADA文件中加载文件

```
//加载一个已经建好的3D模型或场景，会是一个COLLADA文件，后缀名为.dae

let critterURL = NSBundle.mainBundle().URLForResource("Critter", withExtension: "dae")
let critterData = SCNSceneSource(URL: critterURL!, options: nil)
let critterNode = critterData?.entryWithIdentifier("Critter", withClass: SCNNode.self)
if (critterNode != nil) {
    critterNode!.position = SCNVector3(0, 0, -10)
    critterNode?.name = "Critter"
    myScenes.rootNode.addChildNode(critterNode!)
}
```

## 添加物理仿真

物理仿真和SpriteKit中的很像，物理实体的类别有一些不同：

静态实体，从不移动，不受重力影响，但会碰撞。

运动学实体不受物理力的作用，但是它是有实体的，在动画中碰到其他物体会将它们推开。

动态实体受物理力及碰撞影响。

神奇的是SceneKit为我们提供了地板类，它的几何形状就是一个地板

```
//像节点添加物理特性
var critterPhysicsShape: SCNPhysicsShape?
if let geometry = critterNode?.geometry {
    critterPhysicsShape = SCNPhysicsShape(geometry: geometry, options: nil)
}
let critterPhysicsBody = SCNPhysicsBody(type: SCNPhysicsBodyType.Dynamic, shape: critterPhysicsShape)
critterPhysicsBody.mass = 1
critterNode?.physicsBody = critterPhysicsBody

//添加一个地板
let floor = SCNFloor()
let floorNode = SCNNode(geometry: floor)
floorNode.position = SCNVector3(0, -10, 0)
myScenes.rootNode.addChildNode(floorNode)
let floorPhysicsBody = SCNPhysicsBody(type: SCNPhysicsBodyType.Static, shape: SCNPhysicsShape(geometry: floor, options: nil))
floorNode.physicsBody = floorPhysicsBody
```

参考文章：

[http://www.jianshu.com/p/75d0bd650840](http://www.jianshu.com/p/75d0bd650840) 
