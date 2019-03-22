---
title: ARKit入门系列六使用SceneKit旋转问题
date: 2017-11-06 18:39:58
tags: [SceneKit,旋转]
---


旋转模型是经常遇到了，我们之前用CABasicAnimation 可以旋转一个view，其实它也可以旋转一个SCNNode。

首先我们要明白一个概念，每个SCNNode 都有自身的三维坐标系，用CABasicAnimation就是让SCNNode绕自身的三维坐标轴旋转，所以要特别注意是坐标轴，不是这个SCNNode的几何中心。一般SceneKit 的自带的几个几何体的坐标系原点(0,0,0)就是这个它的几何中心，比如说SCNBox；SCNSphere等等，所以看上去跟绕几何中心旋转一模一样。

我们先从Demo 入手，这里是模拟 太阳-地球-月球 天体运动的demo，分以下几点：

1， 地球，月球自转 
2， 月球绕着地球转 
3， 地月系统绕着太阳转

## 先实现地球，月球自转

```
// Rotate the moon
    animation = [CABasicAnimation animationWithKeyPath:@"rotation"];
    animation.duration = 1.5;
    animation.toValue = [NSValue valueWithSCNVector4:SCNVector4Make(0, 1, 0, M_PI * 2)];
    animation.repeatCount = FLT_MAX;
    [_moonNode addAnimation:animation forKey:@"moon rotation"];
```


<!-- more -->

其实SceneKit 框架有自己的动画API ，我们这里地球的旋转用它，让大家了解一下

```
 [_earthNode runAction:[SCNAction repeatActionForever:[SCNAction rotateByX:0 y:2 z:0 duration:1]]];  
  //地球自转
```

##  月球绕地球转（父子SCNNode技巧）

月球需要绕着地球的Y轴旋转，我们知道，只有地球才绕自己的Y轴旋转。所以我们可以让地球带着月球绕自己的Y轴旋转。很简单，将月球add 到地球上：

`[_earthNode addChildNode:_moonNode];`

这是就会实现月球绕着地球转的效果。

但现实中，地球的自转周期跟月球的公转周期是不一样的。所以月球不能添加到地球这个SCNNode 上，我们要重新新建一个SCNNode，位置跟地球一样，把_moonNode添加到这个新建的SCNNode上，然后旋转它。

```
 _moonNode.position = SCNVector3Make(3, 0, 0);

   // Moon-rotation (center of rotation of the Moon around the Earth)
    SCNNode *moonRotationNode = [SCNNode node];

    [moonRotationNode addChildNode:_moonNode];

    // Rotate the moon around the Earth
    CABasicAnimation *moonRotationAnimation = [CABasicAnimation animationWithKeyPath:@"rotation"];
    moonRotationAnimation.duration = 1.5;
    moonRotationAnimation.toValue = [NSValue valueWithSCNVector4:SCNVector4Make(0, 1, 0, M_PI * 2)];
    moonRotationAnimation.repeatCount = FLT_MAX;
    [moonRotationNode addAnimation:animation forKey:@"moon rotation around earth"];
```

## 地月系统绕着太阳转

这个时候地球和月球要绕着太阳转，可以把地月系统看成一个整体。我们创建一个地月系统earthGroupNode，然后将moonRotationNode 和 earthNode 添加进去：

```
[_earthGroupNode addChildNode:_earthNode];
    [_earthGroupNode addChildNode:moonRotationNode];
```

太阳不自转，同理，在太阳同样的位置创建一个SCNNode，让地月系统earthGroupNode随着这个节点旋转

```
   //离太阳的距离
  _earthGroupNode.position = SCNVector3Make(10, 0, 0);
```

```
// Earth-rotation (center of rotation of the Earth around the Sun)
    SCNNode *earthRotationNode = [SCNNode node];
    [_sunNode addChildNode:earthRotationNode];

    // Earth-group (will contain the Earth, and the Moon)
    [earthRotationNode addChildNode:_earthGroupNode];

    // Rotate the Earth around the Sun
    CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"rotation"];
    animation.duration = 10.0;
    animation.toValue = [NSValue valueWithSCNVector4:SCNVector4Make(0, 1, 0, M_PI * 2)];
    animation.repeatCount = FLT_MAX;
    [earthRotationNode addAnimation:animation forKey:@"earth rotation around sun"];
```

## 数学公式旋转

一个点绕另一个点作圆周运动，是不是很熟悉。对，就是我们之前学习的数学知识，这里完全可以用数学知识做。

相关数学知识点： 任意点a(x,y)，绕一个坐标点b(rx0,ry0)逆时针旋转a角度后的新的坐标设为c(x0, y0)，有公式：

```
x0= (x - rx0)*cos(a) - (y - ry0)*sin(a) + rx0 ;

y0= (x - rx0)*sin(a) + (y - ry0)*cos(a) + ry0 ;

```

OK，有这些数学基础，那我们就很好做了，我们让地月系统绕太阳转的效果用数学方法来实现。太阳(sunNode)是b点，地月系统(earthGroupNode)是a点，我们将地月系统添加到太阳里面：

```
[_sunNode addChildNode:_earthGroupNode];
```

那么相对于a点来说，b点的坐标就是(0,0)，然后我们通过计算得到c点，让c点的坐标重新赋值给earthGroupNode 的 position 就可以了。代码如下：

```
 // custom Action

    float totalDuration = 10.0f;        //10s 围绕地球转一圈
    float duration = totalDuration/360; //每隔duration秒去执行一次


    SCNAction *customAction = [SCNAction customActionWithDuration:duration actionBlock:^(SCNNode * _Nonnull node, CGFloat elapsedTime){


        if(elapsedTime==duration){


            SCNVector3 position = node.position;

            float rx0 = 0;    //原点为0
            float ry0 = 0;

            float angle = 1.0f/180*M_PI;

            float x =  (position.x - rx0)*cos(angle) - (position.z - ry0)*sin(angle) + rx0 ;

            float z = (position.x - rx0)*sin(angle) + (position.z - ry0)*cos(angle) + ry0 ;

            node.position = SCNVector3Make(x, node.position.y, z);

        }

    }];

    SCNAction *repeatAction = [SCNAction repeatActionForever:customAction];

    [_earthGroupNode runAction:repeatAction];
```

从上面可以看出我们用了SceneKit 的API SCNAction 去循环计算赋值，其实最主要的就是actionBlock 里面的代码，你也可以完全用线程sleep 和 NSTimer 去实现。

参考文章：

[http://blog.csdn.net/pzhtpf/article/details/51338176](http://blog.csdn.net/pzhtpf/article/details/51338176) 
[http://blog.csdn.net/pzhtpf/article/details/51335125](http://blog.csdn.net/pzhtpf/article/details/51335125) 