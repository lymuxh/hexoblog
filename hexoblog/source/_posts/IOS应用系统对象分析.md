---
title: IOS应用系统对象分析
date: 2016-11-24 09:58:38
tags: [IOS,info.plist,pch,UIApplication,UIWindow,UIViewController]
---


xcode 创建iOS应用后项目里面会包含info.plist、UIApplication、UIWindow、UIViewController文件，为了更好的管理头文件，可以添加pch文件，下面就分别看一下这些文件在应用生命周期中的作用。


# 程序配置文件info.plist

工程的配置中Info选项里面的内容实际上是info.plist文件里面的内容的拷贝，info.plist里面存放了许多关于项目启动参数的配置信息。

<!-- more -->

应该注意：**若想往项目中添加自定义的plist文件，应该使文件名中不包含info**

info.plist修改和其他的plist文件修改方式一样，都是键值对的修改。但是要注意的是当使用Source Code方式打开该文件的时候，键名和用plist方式打开时不一样。就xml源码文件讨论，键名分为以下几种：

Core Foundation Keys

该类的keys的特点是以CF为前缀，用以代表Core Foundation，描述了一些常用的行为项。

Lanch Services Keys


加载服务项，提供了App加载所依赖的配置，描述了app启动的方式选择。

Cocoa Keys

Cocoa框架或Cocoa Touch框架依赖这些keys来标识更高级别的配置项目，如app的main nib文件，主要类。这些key描述影响着Cocoa和Cocoa Touch框架初始化和运行app的运行方式 。

UIKit Keys

描述IOS Apps的行为，每个IOS应用都依赖于Info。plist的keys来与IOS系统通信。Xcode提供了生成的plist文件提供了所有app所需的那些比较重要的keys。 但app可能需要扩展默认的plist来描述更多的信息，如定制app启动后的默认旋转方向，标识app是否支持文件共享等等。

下面是一些常用的键的意义：

1.Localiztion native development region --- CFBundleDevelopmentRegion 本地化相关，如果⽤户所在地没有相应的语言资源，则用这个key的value来作为默认

2.Bundle display name --- CFBundleDisplayName 设置程序安装后显示的名称。应⽤程序名称限制在10-12个字符，如果超出，将被显示缩写名称。

3.Executaule file -- CFBundleExecutable 程序安装包的名称

4.Bundle identidier --- CFBundleIdentidier 该束的唯一标识字符串，该字符串的格式类似 com。yourcompany。yourapp，如果使⽤用模拟器跑你的应用，这个字段没有用处，如果你需要把你的应⽤部署到设备上，你必须⽣成一个证书，⽽在生成证书的时候，在apple的⽹网站上需要增加相应的app IDs。这⾥有一个字段Bundle identidier，如果这个Bundle identidier是一个完整字符串，那么文件中的这个字段必须和后者完全相同，如果app IDs中的字段含有通配符*，那么文件中的字符串必须符合后者的描述。

5.InfoDictionary version --- CFBundleInfoDictionaryVersion Info。plist 格式的版本信息，一般创建后不修改。

6.Bundle name --- CFBundleName 产品名称

7.Bundle OS Type code -- CFBundlePackageType ⽤来标识束类型的四个字母长的代码，一般都是APPL。

8.Bundle versions string， short --- CFBundleShortVersionString ⾯向用户市场的束的版本字符串，应用的版本号。

9.Bundle creator OS Type code --- CFBundleSignature 用来标识创建者的四个字母长的代码。

10.Bundle version --- CFBundleVersion 应⽤程序版本号，每次部署应用程序的一个新版本时， 将会增加这个编号，在app store上用的。

11.Application require iPhone environment -- LSRequiresIPhoneOS 用于指示程序包是否只能运行在iPhone OS 系统上。Xcode自动加入这个键，并将它的值设置为true。您不应该改变这个键的值。

12.Main storybard dile base name -- UIMainStoryboardFile 这是一个字符串，指定应用程序主nib文件的名称。

13.supported interface orientations -- UISupportedInterfaceOrientations 程序默认支持的设备方向。

14.screen interface file base name -- Launch screen 这是一个字符串，指定应用程序启动页nib文件的名称。

15.Status bar tinting parameters -- 定义状态蓝前景色，一般设置根据导航栏默认样式。

# 程序全局头文件pch

pch文件,是prefix header file。是程序运行起来后的全局头文件，当添加了该文件后，不用在其他文件中使用 *#include* 或者是 *#import* 导入该文件，就可以直接使用该文件中的各种内容，比如宏定义和常量和一些公用的类。在xcode5及之前可以直接使用pch文件，在xcode6之后需要在Build Settings中的Apple LLVM 6.0-Language项中的Prefix Header条目中，设置pch文件所在的目录为(SRCROOT)/对应的文件夹名/PrefixHeader.pch，例如在根目录中可以直接设置(SRCROOT)/PrefixHeader.pch。

当程序中的NSLog较多时，会造成很大的性能问题(NSLog是一个IO操作)，对于发布程序是非常不好的，如果依次找到程序中的所有NSLog并将他们删除或者注释，工程项是巨大的，因此可以是用在pch中设置一个条件编译，如果是在调试状态下NSLog为正常的代码，如果在发布状态下则忽略他们。

````objective-c
#ifdef DEBUG
#define NSLog(...); NSLog(__VA_ARGS__);
#else
#define NSLog(...);
#endif
````

在调试状态下 *__VA_ARGS__* 的内容会替换为...中的内容（这是宏定义中提取方法参数中常用的方法），而在发布状态的时候，则直接忽略了NSLog(...)。

ios开发中难免会遇到一些oc与c混编的时候，当使用了pch文件时，c文件是不会识别上面的条件编译和#import等指令的，这时候应该使用另外一个条件编译来避免出现问题：

````
#ifdef __OBJC__
    #import <UIKit/UIKit.h>
    #import <Foundation/Foundation.h>
#endif

````

# 应用程序对象UIApplication

UIApplication是一个应用的单例类，可以做一些应用级别的操作。

1.设置应用图标右上角的消息条数。 --applicationIconBadgeNumber

2.手机联网操作时，状态栏上的菊花等待图标指示器。waiting图标。---networkActivityIndicatorVisible

3.利用UIApplication打开某个资源

    *打开一个网页*

    [app openURL:[NSURL URLWithString:@"https://www.baidu.com"]];

  *打电话*

  [app openURL:[NSURL URLWithString:@"tel://10086"]];

  *发短信*

  [app openURL:[NSURL URLWithString:@"sms://10086"]];

  *发邮件*

  [app openURL:[NSURL URLWithString:@"mailto://1234578@qq.com"]];

4.通过UIApplication管理状态栏

 iOS7开始可以通过两种方式来控制状态栏:

 * 控制器

    ` - (BOOL)prefersStatusBarHidden`

    ` - (UIStatusBarStyle)preferredStatusBarStyle`

  * UIApplication

 如果希望通过UIApplication来管理，步骤如下:

  在Info.plist文件中增加一个配置项

  View controller-based status bar appearance = NO

  然后编写如下代码：// 还可以用方法调用的形式，这样是可以设置动画的。

  app.statusBarHidden = YES;

  app.statusBarStyle=UIStatusBarStyleDefault

# 应用程序委托UIApplicationDelegate

在main函数中进行的设置UIApplication对象的代理。
App容易受到干扰。正在玩游戏，一个电话打过来了。

* 应用程序的生命周期事件(如程序启动和关闭)
* 系统事件(如来电)
* 内存警告
* 推送消息
* … …

** 处理这些干扰事件，就要用到AppDelegate代理对象了。**

 总结: AppDelegate的主要作用就是处理(监听)应用程序本身的各种事件:

* 应用程序启动完毕
* 应用程序进入后台
* 应用程序进入前台
* 内存警告
* 等等, 都是应用程序自身的一些事件

** 要想成为UIApplication的代理对象, 必须遵守:UIApplicationDelegate协议。**

 代理中的若干方法介绍:

`- (BOOL)application: didFinishLaunchingWithOptions:`

 // app第一次启动完毕后就会调用(当程序启动后会显示一张启动图片, 当这个图片显示完毕, 消失后, 就开始调用这个方法)

`- (void)applicationDidEnterBackground:(UIApplication *)application`

 // 当程序进入后台时, 调用该方法。（比如：按了Home键, 或者一个电话打过来了, 当前程序都会进入后台。）
// 在这个方法中可以做一些保存当前程序数据, 暂停程序的操作。

`- (void)applicationWillEnterForeground:(UIApplication *)application`

 // 当程序再次进入前台的时候调用。

`- (void)applicationDidReceiveMemoryWarning:(UIApplication *)application`

 // 当发生内存警告时触发该事件。

`- (void)application:(UIApplication *)application didRegisterUserNotificationSettings:(UIUserNotificationSettings *)notificationSettings NS_AVAILABLE_IOS(8_0) ;`

`- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken `

`- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error NS_AVAILABLE_IOS(3_0);`

  // 进行推送服务注册

`- (UIInterfaceOrientationMask)application:(UIApplication *)application supportedInterfaceOrientationsForWindow:(UIWindow *)window`

 //应用支持屏幕旋转方向，目前旋转有控制器控制

`- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url NS_DEPRECATED_IOS(2_0, 9_0, "Please use application:openURL:options:")`

`- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(nullable NSString *)sourceApplication annotation:(id)annotation NS_DEPRECATED_IOS(4_2, 9_0, "Please use application:openURL:options:")`

 //处理第三方登录回调或启动

程序启动后：

didFinishLaunchingWithOptions-->applicationDidBecomeActive

按home键使程序进入后台：

applicationWillResignActive--->applicationDidEnterBackground

让程序会到主界面：

applicationWillEnterForeground--->applicationDidBecomeActive

在主界面时让程序退出：

applicationDidEnterBackground--->applicationWillTerminate

在后台时退出：

applicationWillTerminate

推送相关：

`- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo`

  iOS10改为使用 UserNotifications Framework's
  
   `-[UNUserNotificationCenterDelegate willPresentNotification:withCompletionHandler:]`
   
   or
  
  `-[UNUserNotificationCenterDelegate didReceiveNotificationResponse:withCompletionHandler:]` 

  for user visible notifications and 
  
  `-[UIApplicationDelegate application:didReceiveRemoteNotification:fetchCompletionHandler:]` 
  
  for silent remote notifications;

`- (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification`

 iOS10改为使用 UserNotifications Framework's 
 
 `-[UNUserNotificationCenterDelegate willPresentNotification:withCompletionHandler:]`
 
  or 
  
  `-[UNUserNotificationCenterDelegate didReceiveNotificationResponse:withCompletionHandler:]`



# 应用显示视图UIWindow

* UIWindow是一种特殊的UIView, UIWindow也是继承自UIView。

* 通常一个app只会有一个UIWindow对象。

* iOS程序启动完毕后，创建的第一个视图控件就是UIWindow，接着创建控制器的view，最后将控制器的view添加到UIWindow上，于是控制器的view就显示在屏幕上了。

* 一个iOS程序之所以能显示到屏幕上，完全是因为它有UIWindow。

* 在文档中找: Cocoa Touch Layer -> UIKit -> Guides -> View Controller Programming Guide for iOS -> View Controller Basics -> 关于UIWindow与控制器View的关系图片。

* 创建过程UIWindow -> UIViewController -> UIView -> 把UIView加到UIWindow对象中。

# 应用界面显示过程

当配置了启动的时候使用main.storyboard或者早期的main.xib的时候, 此时, 当程序启动完毕后, 会自动创建UIWindow, 然后再根据main.storyboard文件或者main.xib创建对应的控制器, 以及控制器中的view. 然后把控制器的View添加到UIWindow上.然后我们就看到界面了。

如果没有为项目配置启动的时候使用哪个storyboard或者早期的main.xib, 那么应用程序启动完毕以后不会创建UIWindow对象, 以及对应的控制器等等.这时就需要自己去创建。

在app代理类的FinishLaunching中

```objective-c
self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];

// 创建一个控制器, 然后把控制器设置给UIWidnow
ViewController *rootViewCotroller = [[ViewController alloc] init];

// 设置hmVc为self.window的根控制器
self.window.rootViewController = rootViewCotroller;

// 把这个(self.window)设置为主窗口, 并且显示出来
[self.window makeKeyAndVisible];

```

需要注意的是：

不要直接把控件添加到UIWindow上, 而是要现创建控制器, 然后向控制器所管理的view中添加子控件, 然后把控制器设置给UIWindow.

原因:

1. UIWindow一直会存在, 直到应用程序退出.所有加到UIWindow中的子控件也就一直会被UIWindow强引用。

2. 如果直接把子控件加到UIWindow中, 那么所有子控件的事件都需要让应用程序代理来监听。实际是应该让对应的控制器来监听这些事件。

3. 当屏幕旋转的时候, UIWindow监听到了这个旋转事件, 然后把它传递给控制器, 控制器再让对应的子控件做旋转.如果要是直接向UIWindow中添加子控件, 那么就没有控制器, 子控件无法监听旋转事件。

# 应用启动流程

**有storyboard、xib**
1. 调用main函数。
2. 调用UIApplicationMain函数。
3. 创建UIApplication对象 、 AppDelegate对象
4. 设置UIApplicatio对象的代理是AppDelegate对象。
5. AppDelegate对象开始监听"系统事件（应用程序的事件）"，进入"事件循环"
6. 程序启动完毕后调用 application: didFinishLaunchingWithOptions:方法。
7. 在application: didFinishLaunchingWithOptions:方法中创建:
* 系统自动创建UIWindow对象。
* 根据Info.plist文件配置(Main Interface),找到需要加载的storyboard文件（Main.storyboard） 或者MainViewController.xib
* 找到Main.storyboard中的Is Initial View Controller或者MainViewController.xib 对应的控制器类, 创建该控制器对象。
* 根据storyboard中的配置, 创建控制器对应的view。
* 设置UIWindow的根控制器(rootViewController)为刚才创建的控制器。
* 显示UIWindow（[self.window makeKeyAndVisible]）。

**纯代码**

1. 调用main函数。
2. 调用UIApplicationMain函数。
3. 创建UIApplication对象 、 AppDelegate对象
4. 设置UIApplicatio对象的代理是AppDelegate对象。
5. AppDelegate对象开始监听"系统事件（应用程序的事件）",进入"事件循环"。
6. 程序启动完毕后调用 application: didFinishLaunchingWithOptions:方法。
7. 在application: didFinishLaunchingWithOptions:方法中创建:
* UIWindow
* 控制器
* 设置UIWindow的根控制器是刚才创建的控制器
* 显示UIWindow

本文章大部分内容参考并摘自[Mike_zh的这篇blog](http://www.cnblogs.com/Mike-zh/p/4430167.html)，主要做复习和汇总了。

参考文章

[iOS屏幕旋转](http://www.jianshu.com/p/d8018006f0b5)

[handleOpenURL](http://www.jianshu.com/p/bb0376d88d0f)
