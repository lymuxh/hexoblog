---
title: iOS获取手机型号系统版本
date: 2017-09-27 19:14:47
tags: [iOS,系统型号,系统版本]
---

新添加判断iPhone X 、iPhone 8、iPhone 8 Plus 

## 手机系统版本

`NSString* phoneVersion = [[UIDevice currentDevice] systemVersion];`

## 手机类型

`NSString* phoneModel = [self iphoneType];`//方法在下面

## 手机系统

`NSString * iponeM = [[UIDevice currentDevice] systemName];`

## 电池电量

`CGFloat batteryLevel=[[UIDevicecurrentDevice]batteryLevel];`

<!-- more -->

## iphone机型

//需要导入头文件：

`#import <sys/utsname.h>`

```- (NSString*)iphoneType {

struct utsname systemInfo;

uname(&systemInfo);

NSString *platform = [NSString stringWithCString:systemInfo.machine encoding:NSASCIIStringEncoding];

if([platform isEqualToString:@"iPhone1,1"])return @"iPhone 2G";

if([platform isEqualToString:@"iPhone1,2"])return @"iPhone 3G";

if([platform isEqualToString:@"iPhone2,1"])return @"iPhone 3GS";

if([platform isEqualToString:@"iPhone3,1"])return @"iPhone 4";

if([platform isEqualToString:@"iPhone3,2"])return @"iPhone 4";

if([platform isEqualToString:@"iPhone3,3"])return @"iPhone 4";

if([platform isEqualToString:@"iPhone4,1"])return @"iPhone 4S";

if([platform isEqualToString:@"iPhone5,1"])return @"iPhone 5";

if([platform isEqualToString:@"iPhone5,2"])return @"iPhone 5";

if([platform isEqualToString:@"iPhone5,3"])return @"iPhone 5c";

if([platform isEqualToString:@"iPhone5,4"])return @"iPhone 5c";

if([platform isEqualToString:@"iPhone6,1"])return @"iPhone 5s";

if([platformisEqualToString:@"iPhone6,2"])return @"iPhone 5s";

if([platform isEqualToString:@"iPhone7,1"])return @"iPhone 6 Plus";

if([platform isEqualToString:@"iPhone7,2"])return @"iPhone 6";

if([platform isEqualToString:@"iPhone8,1"])return @"iPhone 6s";

if([platform isEqualToString:@"iPhone8,2"])return @"iPhone 6s Plus";

if([platform isEqualToString:@"iPhone8,4"])return @"iPhone SE";

if([platform isEqualToString:@"iPhone9,1"])return @"iPhone 7";

if([platform isEqualToString:@"iPhone9,2"])return @"iPhone 7 Plus";

if([platform isEqualToString:@"iPhone10,1"])return @"iPhone 8";

if([platform isEqualToString:@"iPhone10,4"])return @"iPhone 8";

if([platform isEqualToString:@"iPhone10,2"])return @"iPhone 8 Plus";

if([platform isEqualToString:@"iPhone10,5"])return @"iPhone 8 Plus";

if([platform isEqualToString:@"iPhone10,3"])return @"iPhone X";

if([platform isEqualToString:@"iPhone10,6"])return @"iPhone X";

if([platform isEqualToString:@"iPod1,1"])return @"iPod Touch 1G";

if([platform isEqualToString:@"iPod2,1"])return @"iPod Touch 2G";

if([platform isEqualToString:@"iPod3,1"])return @"iPod Touch 3G";

if([platform isEqualToString:@"iPod4,1"])return @"iPod Touch 4G";

if([platform isEqualToString:@"iPod5,1"])return @"iPod Touch 5G";

if([platform isEqualToString:@"iPad1,1"])return @"iPad 1G";

if([platform isEqualToString:@"iPad2,1"])return @"iPad 2";

if([platform isEqualToString:@"iPad2,2"])return @"iPad 2";

if([platform isEqualToString:@"iPad2,3"])return @"iPad 2";

if([platform isEqualToString:@"iPad2,4"])return @"iPad 2";

if([platform isEqualToString:@"iPad2,5"])return @"iPad Mini 1G";

if([platform isEqualToString:@"iPad2,6"])return @"iPad Mini 1G";

if([platform isEqualToString:@"iPad2,7"])return @"iPad Mini 1G";

if([platform isEqualToString:@"iPad3,1"])return @"iPad 3";

if([platform isEqualToString:@"iPad3,2"])return @"iPad 3";

if([platform isEqualToString:@"iPad3,3"])return @"iPad 3";

if([platform isEqualToString:@"iPad3,4"])return @"iPad 4";

if([platform isEqualToString:@"iPad3,5"])return @"iPad 4";

if([platform isEqualToString:@"iPad3,6"])return @"iPad 4";

if([platform isEqualToString:@"iPad4,1"])return @"iPad Air";

if([platform isEqualToString:@"iPad4,2"])return @"iPad Air";

if([platform isEqualToString:@"iPad4,3"])return @"iPad Air";

if([platform isEqualToString:@"iPad4,4"])return @"iPad Mini 2G";

if([platform isEqualToString:@"iPad4,5"])return @"iPad Mini 2G";

if([platform isEqualToString:@"iPad4,6"])return @"iPad Mini 2G";

if([platform isEqualToString:@"iPad4,7"])return @"iPad Mini 3";

if([platform isEqualToString:@"iPad4,8"])return @"iPad Mini 3";

if([platform isEqualToString:@"iPad4,9"])return @"iPad Mini 3";

if([platform isEqualToString:@"iPad5,1"])return @"iPad Mini 4";

if([platform isEqualToString:@"iPad5,2"])return @"iPad Mini 4";

if([platform isEqualToString:@"iPad5,3"])return @"iPad Air 2";

if([platform isEqualToString:@"iPad5,4"])return @"iPad Air 2";

if([platform isEqualToString:@"iPad6,3"])return @"iPad Pro 9.7";

if([platform isEqualToString:@"iPad6,4"])return @"iPad Pro 9.7";

if([platform isEqualToString:@"iPad6,7"])return @"iPad Pro 12.9";

if([platform isEqualToString:@"iPad6,8"])return @"iPad Pro 12.9";

if([platform isEqualToString:@"i386"])return @"iPhone Simulator";

if([platform isEqualToString:@"x86_64"])return @"iPhone Simulator";
returnplatform;
}
```

## 系统版本比较定义

```
#define SYSTEM_VERSION_EQUAL_TO(v)                  ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedSame)

#define SYSTEM_VERSION_GREATER_THAN(v)              ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedDescending)

#define SYSTEM_VERSION_GREATER_THAN_OR_EQUAL_TO(v)  ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] != NSOrderedAscending)
#define SYSTEM_VERSION_LESS_THAN(v)                 ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedAscending)

#define SYSTEM_VERSION_LESS_THAN_OR_EQUAL_TO(v)     ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] != NSOrderedDescending)
```


## 定义主线程执行

```#define dispatch_async_main_safe(block)\
if ([NSThread isMainThread]) {\
block();\
} else {\
dispatch_async(dispatch_get_main_queue(), block);\
}
```



参考网址

[天明依旧-iOS 获取手机型号，系统版本](http://www.jianshu.com/p/02bba9419df8)

