---
title: XCODE报错调试汇总
date: 2016-09-27 18:22:49
tags: [Crash,XCODE]
---

## crash report 获取分析

 在提交新版本到AppStore时候一定要保留每个版本的.dSYM文件。无法定位crash的代码肯定是.dSYM文件和crash report对应的程序版本不一致。

1.由用户反馈的当前版本的crash report可以在iTunesConnect下载。如果是存在设备上的，可以用Organizer导出。

2.打开.crash文件，参考Hardware Model确定故障设备是armv6还是armv7架构。

3.利用.dSYM定位调用栈记录中未确定的offset。

<!-- more -->

比如有如下记录:

　　Last Exception Backtrace:

　　0 CoreFoundation 0x3440a8bf __exceptionPreprocess

　　1 libobjc.A.dylib 0x3465a1e5 objc_exception_throw

　　2 CoreFoundation 0x3440a7b9 +[NSException raise:format:]

　　3 CoreFoundation 0x3440a7db +[NSException raise:format:]

　　4 CoreFoundation 0x3437762b -[__NSCFDictionary setObject:forKey:]

　　5 MyApp 0x000e6f8b 0x1000 + 941963

　　6 MyApp 0x0004140b 0x1000 + 263179

　　...//更多stacktrace

　　使用atos命令和.dSYM确定0x000e6f8b和0x0004140b的对应代码。在命令行中输入：

　　atos -o MyApp.app/MpApp -arch armv7 0x000e6f8b 0x0004140b

　　得到如下结果：

　　-[MyViewController foo:] (in MyApp) (MyViewController.m:457)

　　-[MyDelegate bar:] (in MyApp) (MyDelegate.m:441)

　　这样就确定了调用栈中offset对应的源代码位置。注意如果此时.dSYM文件版本不对，会得到完全无用的错误结果。之后就可以根据所有信息综合分析，确定crash的原因并修正。
　　
## 常见iOS Crash 原因

### 内存管理错误

内存管理是iPhone开发所要掌握的最基本问题，特别是使用引用计数手动管理内存的情况。内存管理错误包括：

内存泄漏：未释放不会再使用对象。比如alloc忘记release,malloc忘记free。可用Xcode的Product菜单下的Analyze功能来解决该问题；

引用出错：引用已经被释放的对象指针。很多“莫名其妙”的Crash都是由于窗体经历的生命周期所导致的（viewDidUnload、viewDidLoad），在iOSSimulator里模拟内存警告就可以解决该问题；

内存警告：App使用的内存超出设备的限制，iOS将强制挂起App，强制挂起iOS是不会记录Crashlog，Flurry也无法记录。内存泄漏、快速/大量的分配内存都可能导致内存警告，这时候应该尽可能的释放不需要的资源。通过Instruments->Allocations里的Heapshot功能能够找出哪些资源未被释放。

UIApplicationMain 的地方：关于EXC_BAD_ACCESS
product->Edit Scheme，在Environment Varibles中 添加 NSZombieEnabled YES 最后结果
关于出现僵尸信号SIGBAT或者EXC_BAD_ACCESS的解决方案[参考解决](http://www.cnblogs.com/zlja/archive/2012/04/12/2444793.html)

performSelector:withObject:withObject:]: message sent to deallocated
问题其实出现在deallocated上，因为controller被设置为autorelease，view被addsubview到某个container上，所以触发事件的时候，发送message到已经dealloc的controller上，导致上述错误。其实以前已经发现过这个问题，就是作为全局变量的string，如果不是是alloc，或者直接（指针）赋值过来的字符串，会不定时的被release掉，造成程序的不稳定

WWDC 2012的Session242 - iOS App Performance_ Memory是专门讨论内存管理这个话题。

### 程序逻辑错误

数组越界、堆栈溢出、并发操作、逻辑错误。扎实的编码基础、严谨细致的工作习惯、清晰的思路可以避免这类错误；

### SDK错误 （部署版本< 编译版本）

这个错误出现的现象是有的设备运行正常，有的会Crash。原因是未找到框架、类、方法、属性。比如：用iOS5.0 SDK编译并运行在iOS4.0的设备上，5.0的Twitter框架在4.0的设备上找不到。这种问题常出现在用苹果新发布的Xcode编译原有的工程。

未找到框架的解决办法是：部署版本>= 编译版本。iOS框架向后兼容做的很棒，部署版本> 编译版本一般不会出现问题。

未找到类、方法、属性的解决办法是：先判断是否存在再使用

`if(NSClassFromString(@"MFMailComposeViewController"))`

`respondsToSelector:`

### 主线程阻塞

主线程阻塞超过10s，iOS将强制挂起App。把长时间的任务放到后台线程去执行，可使用NSThread,NSOperation, dispatch。WWDC2012的Session235 - iOS App Performance_ Responsiveness有详细的介绍。

## 解决Crash

思路是：定位Crash的程序代码，预测Crash原因，寻找解决方案，测试。

有多种方式可以定位Crash的程序代码：

* Debug模式时，iOSSimulator断点测试定位Crash的堆栈；

* 真机连接iTunes查看Crashlog (Debug模式下)；

* 通过Flurry的错误记录查看；

定位之后，就是重新思考程序上下文逻辑，并有理由的预测Crash出现的原因。预测的越多，理解的越深。

寻找解决方案的方法有：

* 浏览苹果官方SDK文档，找出错误原因；

* Google搜索Crash输出的信息，重点查找行业内技术论坛：cocoachina、stackoverflow、iphonedevsdk等；

* 查看历届WWDC的视频、示例代码；

* 在工程里添加环境变量: NSZombieEnabled、NSDebugEnabled，输出有价值的信息；

* 如果未找到任何信息，可以寻求苹果官方论坛、业内技术论坛的帮助；

## 测试

找到解决方案后就需要测试，测试功能输入输出的准确性、程序性能、是否引入新的bug。测试有专业的测试工程师来负责，但开发工程师不能依赖测试工程师来发现问题，尽量独立解决已知存在的问题。

由于Xcode部署工程到真机上比较耗时间，如果可以的话尽可能用iOSSimulator来测试，以减少测试的时间。

建议开发工程师有一个checklist，在产品测试时自己逐一过一下上面常见的问题，这个能够避免大部分Crash。

## 添加收集IOSCrash框架

```objective-c
@interface CatchCrash : NSObject

void uncaughtExceptionHandler(NSException *exception);

@end

@implementation CatchCrash

void uncaughtExceptionHandler(NSException *exception)

{
// 异常的堆栈信息

NSArray *stackArray = [exception callStackSymbols];

// 出现异常的原因

NSString *reason = [exception reason];

// 异常名称

NSString *name = [exception name];

NSString *exceptionInfo = [NSString stringWithFormat:@"Exception reason：%@\nException name：%@\nException stack：%@",name, reason, stackArray];

NSLog(@"%@", exceptionInfo);

NSMutableArray *tmpArr = [NSMutableArray arrayWithArray:stackArray];

[tmpArr insertObject:reason atIndex:0];

//保存到本地  --  当然你可以在下次启动的时候，上传这个log

[exceptionInfo writeToFile:[NSString stringWithFormat:@"%@/Documents/error.log",NSHomeDirectory()]  atomically:YES encoding:NSUTF8StringEncoding error:nil];

}

@end
```

//在appdelegate里添加注册消息处理函数的处理方法

`NSSetUncaughtExceptionHandler(&uncaughtExceptionHandler);`

* 为了能够第一时间发现程序问题，应用程序需要实现自己的崩溃日志收集服务，成熟的开源项目很多，如KSCrash，plcrashreporter，CrashKit等。追求方便省心，对于保密性要求不高的程序来说，也可以选择各种一条龙Crash统计产品，如Crashlytics，Hockeyapp，友盟，Bugly等等。
