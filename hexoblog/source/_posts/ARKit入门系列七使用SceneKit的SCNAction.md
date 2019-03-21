---
title: ARKit入门系列七使用SceneKit的SCNAction
date: 2017-11-06 18:40:39
tags: [SceneKit,SCNAction]
---

# SCNAction

前面关于旋转我们利用CABasicAnimation来实现旋转动画，其实在SceneKit中，有一种更为简单的方法去实现一些基础动画，那就是SCNAction，它的执行对象是SCNNode。

一个简单的例子：
```
SCNAction *shipMoveAction = [SCNAction moveTo:SCNVector3Make(10,10,5) duration:4];

[shipRotationNode runAction:shipMoveAction];
```

上面代码很容易理解shipRotationNode 动画移动到(10,10,5)这个位置，时间间隔为4s。

我们下面简单介绍一下 SCNAction 主要的API：

<!-- more -->

```
+ (SCNAction *)moveByX:(CGFloat)deltaX
                     y:(CGFloat)deltaY
                     z:(CGFloat)deltaZ
              duration:(NSTimeInterval)duration  

 //将node从x,y,z上各移动多少距离

+ (SCNAction *)moveBy:(SCNVector3)delta
             duration:(NSTimeInterval)duration    

 //同上，只不过传入参数为SCNVector3


+ (SCNAction *)moveTo:(SCNVector3)location
             duration:(NSTimeInterval)duration

//将node移动到location这个位置


+ (SCNAction *)rotateByX:(CGFloat)xAngle
                       y:(CGFloat)yAngle
                       z:(CGFloat)zAngle
                duration:(NSTimeInterval)duration

//将node从x,y,z上各旋转多少度


+ (SCNAction *)rotateToX:(CGFloat)xAngle
                       y:(CGFloat)yAngle
                       z:(CGFloat)zAngle
                duration:(NSTimeInterval)duration

//将node从x,y,z上旋转到指定角度


+ (SCNAction *)rotateToX:(CGFloat)xAngle
                       y:(CGFloat)yAngle
                       z:(CGFloat)zAngle
                duration:(NSTimeInterval)duration
         shortestUnitArc:(BOOL)shortestUnitArc

// 同上，与上面的方法区别在于多了shortestUnitArc 这个参数，BOOL值。
//举个例子：我们需要将一个node从 0度旋转到270度，
//如果将shortestUnitArc设置为NO，node会顺时针旋转到270度；
//如果将shortestUnitArc设置为YES，node会逆时针旋转90度到270度，
//即选择最小的旋转角度旋转到特定的度数。默认为NO。


+ (SCNAction *)rotateByAngle:(CGFloat)angle
                  aroundAxis:(SCNVector3)axis
                    duration:(NSTimeInterval)duration

// 沿着特定的轴旋转angle度。前面旋转都是沿x,y,z轴旋转，都是互相垂直的，
//大家有没有想过如何沿着与x轴成45度夹角的方向旋转node？ 
//这个API大家这里留意一下，
//上篇提到的不在X-Z 这个平面旋转，会用这个方法在后面的demo中解决。


+ (SCNAction *)rotateToAxisAngle:(SCNVector4)axisAngle
                        duration:(NSTimeInterval)duration

// SCNVector4(x,y,z,angle)  沿着特定的轴旋转到angle度。
//这里解释一下angle 类似π，如果angle=2,
//我们可不能理解为旋转到2度，而是旋转到2/π*180 度。


+ (SCNAction *)scaleBy:(CGFloat)scale
              duration:(NSTimeInterval)sec

//缩小（放大）多少


+ (SCNAction *)scaleTo:(CGFloat)scale
              duration:(NSTimeInterval)sec

//缩小（放大）到多少


+ (SCNAction *)fadeInWithDuration:(NSTimeInterval)sec

//字面意思可以理解，淡入。将node 的opacity 渐渐变成1


+ (SCNAction *)fadeOutWithDuration:(NSTimeInterval)sec

// 淡出

+ (SCNAction *)fadeOpacityBy:(CGFloat)factor
                    duration:(NSTimeInterval)sec

//将node 的opacity 渐渐变化特定的数值


+ (SCNAction *)fadeOpacityTo:(CGFloat)opacity
                    duration:(NSTimeInterval)sec

//将node 的opacity 渐渐变化到特定的数值


+ (SCNAction *)hide

// 隐藏node


+ (SCNAction *)unhide

//显示node


+ (SCNAction *)removeFromParentNode

//移除node


+ (SCNAction *)playAudioSource:(SCNAudioSource *)source
             waitForCompletion:(BOOL)wait

//播放音频。 waitForCompletion，BOOL值，
//如果为YES Action的duration就是音频的时长；
//如果为NO，可以认为duration 为0。 
//可以去看SCNAudioPlayer 的API.


+ (SCNAction *)group:(NSArray<SCNAction *> *)actions

//group 被用来并发执行多个SCNAction


+ (SCNAction *)sequence:(NSArray<SCNAction *> *)actions

//顺序执行多个SCNAction，上个SCNAction执行结束后，才执行下个SCNAction


+ (SCNAction *)repeatAction:(SCNAction *)action
                      count:(NSUInteger)count

//将一个SCNAction执行count 次


+ (SCNAction *)repeatActionForever:(SCNAction *)action

// 一直执行某个SCNAction


+ (SCNAction *)waitForDuration:(NSTimeInterval)sec

//延迟SCNAction，比如用sequence 顺序执行多个SCNAction时，
//可以给SCNAction a,c  中间添加一个SCNAction b,  
//等a执行结束后，延迟一会，再去执行c


+ (SCNAction *)runBlock:(void (^)(SCNNode *node))block

//自定义SCNAction ，你可以在block 做一些操作


+ (SCNAction *)runBlock:(void (^)(SCNNode *node))block
                  queue:(dispatch_queue_t)queue

//在一个特定的队列中，执行block


+ (SCNAction *)customActionWithDuration:(NSTimeInterval)seconds
                            actionBlock:(void (^)(SCNNode *node,
                                                  CGFloat elapsedTime))block

//上篇数学旋转用到的方法，当这个SCNAction执行时，
//SceneKit 在这个时间间隔内会重复调用actionBlock，
//并将已逝去的时间传给actionBlock


+ (SCNAction *)javaScriptActionWithScript:(NSString *)script
                                 duration:(NSTimeInterval)seconds

//在时间间隔内，执行一段JavaScript代码


- (SCNAction *)reversedAction

//逆转一个已经创建的SCNAction，很好理解，
//相当于CABasicAnimation的autoreverses属性。哪里来的，回哪里去。


@property(nonatomic) NSTimeInterval duration

//SCNAction 的属性，时间间隔，真实时间间隔受speed影响


@property(nonatomic) CGFloat speed

//SCNAction 的属性，速度系数。假设duration 为10，但speed 为2的话，
//就是速度是以前的2倍，实际duration 就为5


@property(nonatomic) SCNActionTimingMode timingMode


//SCNAction 的属性，定时模式,有四个常量值：

Constants
SCNActionTimingModeLinear
//匀速

SCNActionTimingModeEaseIn
//一开始慢，慢慢加快

SCNActionTimingModeEaseOut
//一开始快，逐渐变慢

SCNActionTimingModeEaseInEaseOut
//开始慢慢地,通过中间的时候加速,然后再次放缓
```

## 飞船3D旋转

添加一艘飞船，让它绕着与x轴成45度的方向做圆周运动。

用到的API ：

```
+ (SCNAction *)rotateByAngle:(CGFloat)angle
                  aroundAxis:(SCNVector3)axis
                    duration:(NSTimeInterval)duration
```



参考文章：

[http://blog.csdn.net/pzhtpf/article/details/51353071](http://blog.csdn.net/pzhtpf/article/details/51353071) 

