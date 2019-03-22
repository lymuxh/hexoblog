---
title: UIView 动画
date: 2016-11-30 09:30:31
tags: [UIView,动画]
---

# UIView 动画

在开发APP中，我们会经常用到动画。使用动画会让我们的APP更炫酷，更重要是优化用户体验。在APP开发中实现动画效果有很多种方式，对于简单的应用场景，我们就选择用UIKit提供的动画来实现。

 ## UIView 动画简介

 UIView动画实质上是对CoreAnimation的封装，提供简洁的动画接口。

 <!-- more -->

 UIView动画可以设置的动画属性有:

 大小变化(frame)

 拉伸变化(bounds)

 中心位置(center)

 旋转(transform)

 透明度(alpha)

 背景颜色(backgroundColor)

 拉伸内容(contentStretch)

## UIView动画方法

1.动画开始和结束方法

   `[UIViewbeginAnimations:(nullableNSString*)context:(nullablevoid*)];`

 参数一：动画标识

 参数二：动画上下文，附加参数，大部分情况下，我们设置为nil即可。

 通过UIView 的`setAnimationDelegate:`类方法来设置委托， 并通过 `setAnimationWillStartSelector:`和
`setAnimationDidStopSelector:`方法来指定接收消息的选择器方法。消息处理方法的形式如下：

 ```objective-c
 - (void)animationWillStart:(NSString *)animationID context:(void *)context;
 - (void)animationDidStop:(NSString *)animationID finished:(NSNumber *)finished context:(void
*)context;
```
上面两个方法的animationID 和context 参数和动画块开始时传 `beginAnimations:context:`方法的参数相同：

 animationID - 应用程序提供的字符串，用于标识一个动画块中的动画。

 context - 也是应用程序提供的对象，用于向委托对象传递额外的信息。

 `setAnimationDidStopSelector:`选择器方法还有一个参数—即一个布尔值。如果动画顺利完成，没有被其它动画取消或停止，则该值为YES。
 在设置了代理的情况下，此参数将发送到`setAnimationWillStartSelector`和`setAnimationDidStopSelector`所指定的方法。

 `[UIView commitAnimations];`

 动画提交。

2.动画参数设置

 ```objective-c

 //动画持续时间
[UIView setAnimationDuration:(NSTimeInterval)];
//动画的代理对象
[UIView setAnimationDelegate:(nullableid)];
//设置动画将开始时代理对象执行的SEL(必须先设置代理)
[UIView setAnimationWillStartSelector:(nullableSEL)];
//设置动画结束时代理对象执行的SEL(必须先设置代理)
[UIView setAnimationDidStopSelector:(nullableSEL)];
//设置动画延迟执行的时间
[UIView setAnimationDelay:(NSTimeInterval)];
//设置动画的重复次数
[UIView setAnimationRepeatCount:(float)];
//设置动画的曲线
[UIView setAnimationCurve:(UIViewAnimationCurve)];
```

 UIViewAnimationCurve的枚举值如下：
 UIViewAnimationCurveEaseInOut,//慢进慢出（默认值）
 UIViewAnimationCurveEaseIn,//慢进
 UIViewAnimationCurveEaseOut,//慢出
 UIViewAnimationCurveLinear//匀速

 ```objective-c
 //设置是否从当前状态开始播放动画
 [UIView setAnimationBeginsFromCurrentState:YES];

 ```
   假设上一个动画正在播放，且尚未播放完毕，我们将要进行一个新的动画：

   当为YES时：动画将从上一个动画所在的状态开始播放.

   当为NO时：动画将从上一个动画所指定的最终状态开始播放.

 ```objective-c
 //设置动画是否继续执行相反的动画
[UIView setAnimationRepeatAutoreverses:(BOOL)];
//是否禁用动画效果（对象属性依然会被改变，只是没有动画效果）
[UIView setAnimationsEnabled:(BOOL)];
//设置视图的过渡效果
[UIView setAnimationTransition:(UIViewAnimationTransition)forView:(nonnullUIView*)cache:(BOOL)];

 ```

 第一个参数：UIViewAnimationTransition的枚举值如下

 UIViewAnimationTransitionNone,//不使用动画
 UIViewAnimationTransitionFlipFromLeft,//从左向右旋转翻页
 UIViewAnimationTransitionFlipFromRight,//从右向左旋转翻页
 UIViewAnimationTransitionCurlUp,//从下往上卷曲翻页
 UIViewAnimationTransitionCurlDown,//从上往下卷曲翻页

 第二个参数：需要过渡效果的View

 第三个参数：是否使用视图缓存，YES：视图在开始和结束时渲染一次；NO：视图在每一帧都渲染

 `-(void)removeAllAnimations;`

  删除所有动画层，注意：层指的layer。例如：
  `[_imageView.layer removeAllAnimations];`


## UIView Block动画

 iOS4.0以后，增加了Block动画块，提供更简洁的方式来实现动画。

   * 最简洁的Block动画：包含时间和动画

  ```objective-c
  [UIView animateWithDuration:(NSTimeInterval)//动画持续时间
animations:^{
//执行的动画
}];
  ```
   * 带有动画完成回调的Block动画

   ```objective-c
   [UIView animateWithDuration:(NSTimeInterval)//动画持续时间
animations:^{
//执行的动画
}completion:^(BOOLfinished){
//动画执行完毕后的操作
}];
   ```

   * 可设置延迟时间和过渡效果的Block动画

   ```objective-c
   [UIView animateWithDuration:(NSTimeInterval)//动画持续时间
delay:(NSTimeInterval)//动画延迟执行的时间
options:(UIViewAnimationOptions)//动画的过渡效果
animations:^{
//执行的动画
}completion:^(BOOLfinished){
//动画执行完毕后的操作
}];
   ```

   UIViewAnimationOptions的枚举值如下，可组合使用：

   UIViewAnimationOptionLayoutSubviews//进行动画时布局子控件

   UIViewAnimationOptionAllowUserInteraction//进行动画时允许用户交互

   UIViewAnimationOptionBeginFromCurrentState//从当前状态开始动画

   UIViewAnimationOptionRepeat//无限重复执行动画

   UIViewAnimationOptionAutoreverse//执行动画回路

   UIViewAnimationOptionOverrideInheritedDuration//忽略嵌套动画的执行时间设置

   UIViewAnimationOptionOverrideInheritedCurve//忽略嵌套动画的曲线设置

   UIViewAnimationOptionAllowAnimatedContent//转场：进行动画时重绘视图

   UIViewAnimationOptionShowHideTransitionViews//转场：移除（添加和移除图层的）动画效果

   UIViewAnimationOptionOverrideInheritedOptions//不继承父动画设置

   UIViewAnimationOptionCurveEaseInOut//时间曲线，慢进慢出（默认值）

   UIViewAnimationOptionCurveEaseIn//时间曲线，慢进

   UIViewAnimationOptionCurveEaseOut//时间曲线，慢出

   UIViewAnimationOptionCurveLinear//时间曲线，匀速

   UIViewAnimationOptionTransitionNone//转场，不使用动画

   UIViewAnimationOptionTransitionFlipFromLeft//转场，从左向右旋转翻页

   UIViewAnimationOptionTransitionFlipFromRight//转场，从右向左旋转翻页

   UIViewAnimationOptionTransitionCurlUp//转场，下往上卷曲翻页

   UIViewAnimationOptionTransitionCurlDown//转场，从上往下卷曲翻页

   UIViewAnimationOptionTransitionCrossDissolve//转场，交叉消失和出现

   UIViewAnimationOptionTransitionFlipFromTop//转场，从上向下旋转翻页

   UIViewAnimationOptionTransitionFlipFromBottom//转场，从下向上旋转翻页

   * Spring动画

    iOS7.0后新增Spring动画，Spring 动画是 线性动画或者 ease-out 动画的理想替代品，iOS系统动画大部分采用SpringAnimation，此外，Spring动画不只可对位置使用，它适用于所有可被添加动画效果的属性。

  ```objective-c
  [UIView animateWithDuration:(NSTimeInterval)//动画持续时间
delay:(NSTimeInterval)//动画延迟执行的时间
usingSpringWithDamping:(CGFloat)//震动效果，范围0~1，数值越小震动效果越明显
initialSpringVelocity:(CGFloat)//初始速度，数值越大初始速度越快
options:(UIViewAnimationOptions)//动画的过渡效果
animations:^{
//执行的动画
}
completion:^(BOOLfinished){
//动画执行完毕后的操作
}];

  ```

   * keyFrame 动画

   UIView动画已经具备高级的方法来创建动画，而且可以更好的理解和构建动画。IOS7之后苹果新添加了一个animateKeyframesWithDuration的方法，我们可以使用它创建更多更复杂的动画效果，而不需要使用到核心动画。

   **关键帧动画，支持属性关键帧，不支持路径关键帧**

   ```objective-c
   [UIView animateKeyframesWithDuration:(NSTimeInterval)//动画持续时间
delay:(NSTimeInterval)//动画延迟执行的时间
options:(UIViewKeyframeAnimationOptions)//动画的过渡效果
animations:^{
//执行的关键帧动画
}
completion:^(BOOLfinished){
//动画执行完毕后的操作
}];
   ```

   UIViewKeyframeAnimationOptions的枚举值如下，可组合使用：

   UIViewAnimationOptionLayoutSubviews//进行动画时布局子控件

   UIViewAnimationOptionAllowUserInteraction//进行动画时允许用户交互

   UIViewAnimationOptionBeginFromCurrentState//从当前状态开始动画

   UIViewAnimationOptionRepeat//无限重复执行动画

   UIViewAnimationOptionAutoreverse//执行动画回路

   UIViewAnimationOptionOverrideInheritedDuration//忽略嵌套动画的执行时间设置

   UIViewAnimationOptionOverrideInheritedOptions//不继承父动画设置

   UIViewKeyframeAnimationOptionCalculationModeLinear//运算模式:连续

   UIViewKeyframeAnimationOptionCalculationModeDiscrete//运算模式:离散

   UIViewKeyframeAnimationOptionCalculationModePaced//运算模式:均匀执行

   UIViewKeyframeAnimationOptionCalculationModeCubic//运算模式:平滑

   UIViewKeyframeAnimationOptionCalculationModeCubicPaced//运算模式:平滑均匀

  增加关键帧的方法

  ```objective-c
  [UIView addKeyframeWithRelativeStartTime:(double)//动画开始的时间（占总时间的比例）
relativeDuration:(double)//动画持续时间（占总时间的比例）
animations:^{
//执行的动画
}];

  ```
   * 转场动画

    从旧视图到新视图的动画效果

    ```objective-c
    [UIView transitionFromView:(nonnullUIView*)
toView:(nonnullUIView*)
duration:(NSTimeInterval)
options:(UIViewAnimationOptions)
completion:^(BOOLfinished){
//动画执行完毕后的操作
}];
    ```

    在该动画过程中，fromView会从父视图中移除，并讲toView添加到父视图中，注意转场动画的作用对象是父视图（过渡效果体现在父视图上）。

    调用该方法相当于执行下面两句代码：

    ```objective-c
    [fromView.superview addSubview:toView];
    [fromView removeFromSuperview];
    ```

    单个视图的过渡效果

    ```objective-c
    [UIView transitionWithView:(nonnullUIView*)
    duration:(NSTimeInterval)
    options:(UIViewAnimationOptions)
    animations:^{
    //执行的动画
    }
    completion:^(BOOLfinished){
    //动画执行完毕后的操作
    }];
    ```

  # UIView 动画实例

   1.普通动画

  下面三段代码都实现了相同的视图frame的变化，不同之处只在于其延迟时间、过渡效果和结束回调。


   ```objective-c
    -(void)blockAni{
[UIView animateWithDuration:1.0 animations:^{
self.cartCenter.frame=self.centerShow.frame;
}];
}

-(void)blockAni{
[UIView animateWithDuration:1.0 animations:^{
self.cartCenter.frame=self.centerShow.frame;
}completion:^(BOOL finished){
NSLog(@"动画结束");
}];
}

-(void)blockAni{
[UIView animateWithDuration:1.0 delay:1.0 options:UIViewAnimationOptionCurveEaseInOut animations:^{
self.cartCenter.frame=self.centerShow.frame;
}completion:^(BOOL finished){
NSLog(@"动画结束");
}];
}
    ```

   2.Spring动画

   ```objective-c
   -(void)blockAni{
[UIView animateWithDuration:1.0 delay:0.f usingSpringWithDamping:0.5 initialSpringVelocity:5.0 options:UIViewAnimationOptionCurveLinear animations:^{
self.cartCenter.frame=self.centerShow.frame;
}completion:^(BOOL finished){
NSLog(@"动画结束");
}];
}

   ```

   3.keyFrames动画

  这里以实现视图背景颜色变化（红-绿-蓝-紫）的过程来演示关键帧动画。

   ```objective-c
   -(void)blockAni{
self.centerShow.image=nil;
[UIView animateKeyframesWithDuration:9.0 delay:0.f options:UIViewKeyframeAnimationOptionCalculationModeLinear animations:^{
[UIView addKeyframeWithRelativeStartTime:0.f relativeDuration:1.0/4 animations:^{
self.centerShow.backgroundColor=[UIColor colorWithRed:0.9475 green:0.1921 blue:0.1746 alpha:1.0];
}];
[UIView addKeyframeWithRelativeStartTime:1.0/4 relativeDuration:1.0/4 animations:^{
self.centerShow.backgroundColor=[UIColor colorWithRed:0.1064 green:0.6052 blue:0.0334 alpha:1.0];
}];
[UIView addKeyframeWithRelativeStartTime:2.0/4 relativeDuration:1.0/4 animations:^{
self.centerShow.backgroundColor=[UIColor colorWithRed:0.1366 green:0.3017 blue:0.8411 alpha:1.0];
}];
[UIView addKeyframeWithRelativeStartTime:3.0/4 relativeDuration:1.0/4 animations:^{
self.centerShow.backgroundColor=[UIColor colorWithRed:0.619 green:0.037 blue:0.6719 alpha:1.0];
}];
}completion:^(BOOL finished){
NSLog(@"动画结束");
}];
}
   ```

4.转场动画

  单个视图转场

   ```objective-c
   -(void)blockAni{
   [UIView transitionWithView:self.centerShow duration:1.0 options:UIViewAnimationOptionTransitionCrossDissolve animations:^{
   self.centerShow.image=[UIImage imageNamed:@"Service"];
   }completion:^(BOOL finished){
   NSLog(@"动画结束");
   }];
   }
   ```

   多个视图转场

   ```objective-c
   -(void)blockAni{
UIImageView* newCenterShow =[[UIImageView alloc]initWithFrame:self.centerShow.frame];
newCenterShow.image=[UIImage imageNamed:@"Service"];
[UIView transitionFromView:self.centerShow toView:newCenterShow duration:1.0 options:UIViewAnimationOptionTransitionFlipFromLeft completion:^(BOOL finished){
NSLog(@"动画结束");
}];
}
   ```

   UIView实现动画的方法有很多种。简单的动画效果你可以随意写。比较复杂的动画可以选用KeyFrames。

参考文档：

[http://www.cnblogs.com/superYou/p/4634422.html](http://www.cnblogs.com/superYou/p/4634422.html)
