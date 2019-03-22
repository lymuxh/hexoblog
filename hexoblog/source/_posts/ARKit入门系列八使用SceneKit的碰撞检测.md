---
title: ARKit入门系列八使用SceneKit的碰撞检测
date: 2017-11-06 18:41:19
tags: [SceneKit , 碰撞检测]
---


2D中的碰撞检查大家都能能理解，涉及到的数学知识并不复杂。但在3D中碰撞检测要涉及到更复杂的数学知识了，很多人数学功底不好的同学就犯难了，我也是。幸好SceneKit框架 提供了方便的方法去判断碰撞检测。

要想让SCNNode 模拟碰撞检测，首先要设置它的Physics Body，Physics Body有三种类型： 

 * Dynamic：动态的物体，受力的影响(applyForce)。 
 * Static： 静态的物体，不受力的影响。 
 * Kinemat：这种类型比较特殊，你可以直接移动，旋转它，在空间变换中，会对Dynamic的物体产生力的影响。

```
 //示例1
 rootWallNode.physicsBody = [SCNPhysicsBody dynamicBody];
```

<!-- more -->

```
//示例2
  SCNPhysicsShape *shape = [SCNPhysicsShape shapeWithGeometry:[SCNBox boxWithWidth:5 height:5 length:5 chamferRadius:0] options:nil];
 self.cameraNode.physicsBody = [SCNPhysicsBody bodyWithType:SCNPhysicsBodyTypeKinematic shape:shape];
 ```

我们来解释一下上面的代码 示例1 创建一个dynamicBody，形状与node 的一致。示例2 创建一个 正方体的形状，意思就是大体包住node，对精度要求不高的可以使用示例2，示例2相对于示例1所消耗的性能也小。

当然SceneKit物理引擎为了提高性能，并不是所有设置Physics Body 的node都去检测碰撞，你也只需要某两个node碰撞的事件。所以你需要指定那几个node碰撞时让物理引擎告诉你。

比如说一个房间有一张椅子和一张桌子，你可以移动椅子，你想检测椅子跟桌子，墙的碰撞。 

## 标记

 我们首先需要给node标记各自的身份：

```
//我们建立枚举，让可读性高一点
typedef enum : NSUInteger {
    CollisionDetectionMaskChair = 1,
    CollisionDetectionMaskDesk = 2,
    CollisionDetectionMaskWall = 4
} CollisionDetectionMask;

self.chairNode.physicsBody.categoryBitMask = CollisionDetectionMaskChair;

self.deskNode.physicsBody.categoryBitMask = CollisionDetectionMaskDesk;

self.wallNode.physicsBody.categoryBitMask = CollisionDetectionMaskWall;
```

## 设置碰撞检测

接下来我们告诉物理引擎当椅子与桌子，墙发生碰撞告诉我，设置如下：

```
self.chairNode.physicsBody.collisionBitMask = CollisionDetectionMaskDesk|CollisionDetectionMaskWall;

self.chairNode.physicsBody.contactTestBitMask = CollisionDetectionMaskDesk|CollisionDetectionMaskWall;
```

而桌子，墙 的碰撞对象都是椅子

```
self.deskNode.physicsBody.collisionBitMask = CollisionDetectionMaskChair;

self.deskNode.physicsBody.contactTestBitMask = CollisionDetectionMaskChair;

self.wallNode.physicsBody.collisionBitMask = CollisionDetectionMaskChair;

self.wallNode.physicsBody.contactTestBitMask = CollisionDetectionMaskChair;
```

##  设置代理

 接下来我们就要实现 SCNPhysicsContactDelegate 的代理方法

```
scene.physicsWorld.contactDelegate = self;
```

最主要的三个代理方法如下，从字面意思很好理解，就不一一介绍了

```
#pragma mark SCNPhysicsContactDelegate

- (void)physicsWorld:(SCNPhysicsWorld *)world didBeginContact:(SCNPhysicsContact *)contact{
   //开始碰撞
}
- (void)physicsWorld:(SCNPhysicsWorld *)world didUpdateContact:(SCNPhysicsContact *)contact{

    //得到两个碰撞的node
    SCNNode *nodeA = contact.nodeA;
    SCNNode *nodeB = contact.nodeB;

  //  SCNVector3 contactPoint = contact.contactPoint;
  //碰撞点

 if(nodeA.physicsBody.categoryBitMask == CollisionDetectionMaskChair)                      {

   // 做一些事情
}


}
- (void)physicsWorld:(SCNPhysicsWorld *)world didEndContact:(SCNPhysicsContact *)contact{
   //结束碰撞
}
```

参考文章：
[http://blog.csdn.net/pzhtpf/article/details/52882568](http://blog.csdn.net/pzhtpf/article/details/52882568) 