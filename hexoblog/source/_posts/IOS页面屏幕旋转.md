---
title: IOS页面屏幕旋转
date: 2016-11-24 15:51:13
tags: [iOS,屏幕旋转]
---
IOS应用在使用过程中手机的方向不一定一直是正面竖直状态的，交互好的应用会根据手机的方向动态的调整界面的布局。Apple对手机app的多方向适配没有强制要求，对IPAD应用要求必须适配竖直和水平状态。

# 全局屏幕旋转适配
1.在AppDelegate回调中添加

  `- (UIInterfaceOrientationMask)application:(UIApplication *)application supportedInterfaceOrientationsForWindow:(nullable UIWindow *)window `

2.通过info.plist文件进行配置

# 单个ViewController屏幕旋转适配

<!-- more -->

## 自动旋转

在iOS6之前的版本中，通常使用 shouldAutorotateToInterfaceOrientation 来单独控制某个UIViewController的方向，需要哪个viewController支持旋转，只需要重写shouldAutorotateToInterfaceOrientation方法。
但是iOS 6里屏幕旋转改变了很多，之前的 shouldAutorotateToInterfaceOrientation 被列为 DEPRECATED 方法，查看UIViewController.h文件也可以看到：

```objective-c
- (BOOL)shouldAutorotateToInterfaceOrientation:(UIInterfaceOrientation)toInterfaceOrientation NS_DEPRECATED_IOS(2_0, 6_0);
```

在iOS6中新添加了两个方法

```objective-c
- (BOOL)shouldAutorotate;
- (NSUInteger)supportedInterfaceOrientations;
- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation;
```
除了重写这个3个方法，另外还有一个硬性条件，需要在Info.plist文件里面添加程序支持的所有方向。

单纯重写这三个方法没用，因为文档发现这个需要设置在根试图

![img](/images/viewControllerAutorotate.jpg)


对于以下方法仅对deploy target大于等于iOS6的工程有效，如果需要支持iOS5（默哀），请pass。

在info.plist中设置方向，包含你需要的所有方向，比如，UpSideDown和LandScapeLeft；

继承UITabBarController,override以下三个方法

```objective-c
 - (BOOL)shouldAutorotate
{
    return [self.selectedViewController shouldAutorotate];
}

 - (NSUInteger)supportedInterfaceOrientations
{
    return [self.selectedViewController supportedInterfaceOrientations];
}

 - (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation
{
    return [self.selectedViewController preferredInterfaceOrientationForPresentation];
}

```

继承UINavigationController，override和UITabBarController中相同的方法，将selectedViewController改为topViewController

```objective-c
 - (BOOL)shouldAutorotate
{
    return [self.topViewController shouldAutorotate];
}

 - (NSUInteger)supportedInterfaceOrientations
{
    return [self.topViewController supportedInterfaceOrientations];
}

 - (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation
{
    return [self.topViewController preferredInterfaceOrientationForPresentation];
}

```

在真正实现界面的ViewController里，override上面这三个方法，override规则如下：
preferredInterfaceOrientationForPresentation表示viewController初始显示时的方向；
supportedInterfaceOrientations是在该viewController中支持的所有方向；
shouldAutorotate表示是否允许旋屏。

### 旋转处理流程说明

首先，对于任意一个viewController，iOS会以info.plist中的设置和当前viewController的preferredInterfaceOrientationForPresentation和supportedInterfaceOrientations三者支持的方法做一个交运算，若交集不为空，则以preferredInterfaceOrientationForPresentation为初始方向，交集中的所有方向均支持，但仅在shouldAutorotate返回YES时，允许从初始方向旋转至其他方向。若交集为空，进入viewController时即crash，错误信息中会提示交集为空。

其次，UINavigationController稍有些特别，难以用常规API做到同一个naviVC中的ViewController在不同方向间自如地切换。(如果去SO之类的地方搜索，会找到一个present empty viewController and then dismiss it之类的hacky trick，不太建议使用)，如果要在横竖屏间切换，建议使用presentXXX方法。

再次，AppDelegate中有一个委托方法可以动态的设置应用支持的旋转方向，且此委托的返回值会覆盖info.plist中的固定设置。使用该方法的便利之处不言自明，但缺点是搞明白当前哪个ViewController即将要被显示，很可能会导致耦合增加。

## 手动旋转

 1. 通过设置UIDevice的orientation

```objective-c
if ([[UIDevice currentDevice] respondsToSelector:@selector(setOrientation:)]) {
     [[UIDevice currentDevice] performSelector:@selector(setOrientation:) withObject:(id)UIInterfaceOrientationPortrait];
   }
```

 2. 通过view的transform来旋转界面

 ```objective-c
 self.view.transform = CGAffineTransformMakeRotation(M_PI/2)
 ```
  只需要改变self.view.transform，那么self.view的所有subview都会跟着自动变；其次因为方向变了，所以self.view的大小需要重新设置，不要使用self.view.frame，而是用bounds。

 注意：

 如果shouldAutorotate 返回YES的话，下面设置setStatusBarOrientation 是不管用的！setStatusBarOrientation只有在shouldAutorotate 返回NO的情况下才管用！


参考列表：

[总结iOS App开发中控制屏幕旋转的几种方式](http://blog.csdn.net/li_shuang_ls/article/details/51792578)

[iOS实现屏幕旋转](http://www.cnblogs.com/lear/p/5051818.html)

[iOS指定屏幕旋转](http://www.jianshu.com/p/d8018006f0b5)
