---
title: UIViewController生命周期
date: 2016-11-24 15:53:39
tags: [UIViewController,控制器生命周期]
---

就iOS开发来说，UIViewController就最核心的类型之一。而iOS的整个UI开发的核心思想也是MVC的架构，从UIViewController的命名就可以看出它在MVC中所扮演的角色，那就是Controller啦。

# 创建控制器的三种方式

 1. alloc init方式纯代码创建

  ```objective-c
// 1.创建window self.window是强指针
self.window = [[UIWindow alloc]initWithFrame:[UIScreen mainScreen].bounds];
// 2.创建控制器，并设置为window的根控制器
MKOneViewController *oneViewController = [[MKOneViewController alloc]init];
self.window.rootViewController = oneViewController;
// 3.设置self.window为主窗口, 并显示
[self.window makeKeyAndVisible];
```
<!-- more -->

 2. xib文件创建

  ```objective-c
是让整个xib文件中的控件都让controller管理，xib中没有控制器，而是设置xib的files owner为指定的控制器，控制器用initWithNib创建
// 1.创建window
self.window = [[UIWindow alloc]initWithFrame:[UIScreen mainScreen].bounds];
// 2.使用xib加载控制器，并设置为window的根控制器
MKThreeViewController *threeViewController = [[MKThreeViewController alloc]initWithNibName:@"MKThreeView" bundle:nil];
self.window.rootViewController = threeViewController;
// 3.设置self.window为主窗口，并显示
[self.window makeKeyAndVisible];
```

 3. storyboard文件来创建

 ```objective-c
// 1.创建window
self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
// 2.通过storyboard加载控制器，并将initialViewController设置为window的根控制器
UIStoryboard *sb = [UIStoryboard storyboardWithName:@"Two" bundle:nil]; // sb加载
MKTwoViewController *twoViewController = [sb instantiateInitialViewController]; // 用sb初始化initialViewController
self.window.rootViewController = twoViewController;
// 3.设置window为主窗口并显示
[self.window makeKeyAndVisible];
注意如果想加载的不是initialViewController，而是sb文件中的其他controller，根据控制器的Storyboard ID来创建：
[storyboard instantiateViewControllerWithIdentifier:@"vmid"];
```

**使用xib需要注意的是：**

如果创建控制器的时候, 没有明确指定xib文件(也就是使用这样的代码创建[[MKThreeViewController alloc]init]), 那么默认系统回去查找一个与控制器名字一样的(但是去掉后缀Controller，也就是MKThreeView.xib)的xib文件， 如果找了则使用这个xib中的view作为控制器的默认view(前提是已经将view连线，即使没有设置files owner也是可以的)。

如果找不到则尝试去找一个与控制器名字一模一样的xib文件(MKThreeViewController.xib)， 然后使用这个xib文件中的view作为控制器默认的view。如果都找不到, 那么就创建一个透明的view（空的view）。

建议将这样的xib起名为带controller后缀的，因为它的作用就相当于一个controller，类似于sb中的一个controller。

# 控制器view的创建过程

控制器的负责它管理的view的创建，view到底是如何进行创建的呢,一般来说view的创建和控制器的创建方式是有关系的，控制器的view有以下几种创建的方式：
1. 通过storyboard创建, 创建完控制器后, 自动调用loadView方法,创建控制器的view。
** 此时自定义的控制器, 因为没有"重写"("实现")loadView方法, 所以loadView方法内部就是根据storyboard文件中的view来创建View的。

2. 通过xib文件创建, 创建完控制器后, 自动调用loadView创建控制器的view。
** 此时自定义的控制器, 因为没有"重写"("实现")loadView方法, 所以loadView方法内部就是根据xib文件中的view来创建View的。

3. 通过重写（实现）UIViewController的loadView方法重写。(自己通过代码来创建控制器的View)
*** 控制器的loadView方法就是用来自定义View的。
** 如果要为控制器自定义View, 要写在loadView中, 不要写在viewDidLoad中。
*** 如果重写（实现）loadView方法中调用了[super loadView], 那么依然会使用默认的方式来加载。
*** 重写ViewController的loadView方法，用来自定义View，当自定义view的时候, 就不要调用[super loadView]了。
[super loadView];相当于执行了一下代码

```objective-c
if (是否是根据storyboard来创建的控制器) {
   self.view = storyboard中的控制器中的view;
} else if (xib) {
   self.view = xib中的view;
} else {
   self.view = 透明的一个view
}

```
 注意：

 无论是通过加载xib创建view、storyboard创建view, 最终都依赖于loadView方法来创建。这个方法确定了最终的view。

修改了项目文件（比如:xxx.xib等，要先Product -> Clean, 然后把软件从模拟器中卸载, 然后再运行。）

** 控制器的loadView方法什么时候调用?**

在需要用到控制器的view的时候才调用（当调用UIWindow对象的makeKeyAndVisible方法时，就需要显示该view了, 此时就表示用到View了。）, 这个就叫做"延迟加载"。 比如当使用self.view.backgroundColor , 要设置控制器的view的背景色时（这时需要用到view了）, 那么此时才会开始创建该控制器的view, 也就是说要调用loadView方法了。所以, 如果在loadView方法内部调用self.view.backgroundColor, 就发生死循环了。

 可以通过调用控制器的self.isViewLoaded 方法来判断当前控制器的view是否已经加载完毕了。

并且当控制器的view加载完毕后, 会调用viewDidLoad方法(系统自己调用)。

# Controller的view的生命周期

下面是代表控制器视图的生命周期的7个方法
viewDidLoad

viewWillAppear

viewDidAppear

viewWillDisappear

viewDidDisappear

viewWillUnload

viewDidUnload

下面是这些方法的调用顺序图：

假设有两个控制器：

程序启动后：OneController的viewDidLoad--->OneController的viewWillAppear--->OneController的viewDidAppear

跳转到第二个界面：Two的viewDidLoad--->One的viewWillDisapear--->Two的viewWillAppear----> One的viewDidDisappear--->Two的viewDidAppear

返回第一个界面：Two的viewWillDisapear--->One的viewWillAppear----> Two的viewDidDisappear--->One的viewDidAppear （one的view不需再次创建）

再次跳到第二个界面：和第一次一样，two的view需要创建。

# 导航控制器的使用

导航控制器本身和其他控制器没什么太大区别，但是导航控制器可以负责页面的跳转，用来管理一组控制器，这些被管理的控制器成为它的子控制器(一定和子类的概念区别开)

** 使用步骤:

1> 创建、初始化一个导航控制器: UINavigationController.

2> 设置UIWindow的rootViewController为UINavigationController.

3> 通过调用push方法添加子控制器到UINavigationController。

** 注意：谁是最后一个push进来的，当前显示的就是哪个ViewController**，在

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions中
UINavigationController *nav = [[UINavigationController alloc]init];
self.window.rootViewController = nav; // 设置为window的根控制器

MKOneViewController *oneVC = [[MKOneViewController alloc]init];
[nav pushViewController:oneVC animated:YES]; // 将页面跳转到oneVC的view

```

一般的应用程序中，如果有导航控制，会在创建导航控制器的时候指定它的初始控制器，而不是使用push的方式将初始控制器入栈push：

```objective-c
OneViewController *oneVC = [[OneViewController alloc]init];
UINavigationController *nav = [[UINavigationController alloc]initWithRootViewController:oneVC];
self.window.rootViewController = nav; NavigationController以下几个方法也比较常用：
[nav popViewControllerAnimated:YES]; // 弹出栈顶的控制器， 返回

[nav popToRootViewControllerAnimated:YES]; // 依次弹出栈顶控制器，直到显示根控制器的view
```

要想设置导航条的一些属性，需在导航控制器的子控制器中分别设置：
在每个控制器的viewDidLoad方法中设置当前控制器的navigationItem属性。
navigationItem属性的具体内容:

* title属性
* titleView属性
* leftBarButtonItem 属性：只能设置左上角的一个按钮
* leftBarButtonItems属性: 可以设置左上角有多个按钮
* rightBarButtonItem属性：只能设置右上角的一个按钮
* rightBarButtonItems属性： 可以设置右上角有多个按钮
* backBarButtonItem属性: 设置下一个控制器, 左上角的按钮。默认情况下，该按钮文字与上一个控制器的title文字相同。

**如果使用storyboard设置导航栏和它的子控制器，需要注意的是：导航控制器向子控制器连线的时候选择的是relationship segue：view controllers。**

# UITabBarController的使用

基本上与navigationController的创建方式一样,添加子控件的时候可以使用
viewControllers属性或者setViewControllers:方法添加一个控制器数组
但是不能使用childViewController属性，这是一个只读的属性,下面是一个简单的使用例子：

```objective-c
// 1.创建self.window
self.window = [[UIWindow alloc]initWithFrame:[UIScreen mainScreen].bounds];
// 2.创建tabBarController控制器并设置它的子控制器，同时设置为window的根控制器
UITabBarController *tab = [[UITabBarController alloc]init];

OneViewController *one = [[OneViewController alloc]init];
TwoViewController *two = [[TwoViewController alloc]init];
ThreeViewController *three = [[ThreeViewController alloc]init];

//    [tab setViewControllers:@[one, two, three]];
tab.viewControllers = @[one, two, three];

self.window.rootViewController = tab;
// 3.将window设置为主界面并显示
[self.window makeKeyAndVisible];
```

通过storyboard设置导航栏，是在导航栏显示的时候将所有的按钮都显示了出来，而不像针对每个子控制器设置的导航那样延迟加载。 注意连线时，选择的是relationship segue。

# UITabBarController->UINavigationController->UIViewController 主流框架

目前的大多数程序都采用了TabBarController>>NavigationController>>ViewController主流框架，window的根控制器是一个UITabBarController，其子控制器为多个UINavigationController，每个NavigationController可以segue或者modal出其他的控制器。

本文章大部分内容参考并摘自[Mike_zh的这篇blog](http://www.cnblogs.com/Mike-zh/p/4430616.html)，主要做复习和汇总了。
