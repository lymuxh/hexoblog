---
title: iOS延时执行的几种方式
date: 2017-09-27 19:32:04
tags: [iOS,延时执行,NSTimer,GCD]
---


延时执行方法定义如下

`-(void)delayMethod
{
    NSLog(@"delayMehtod run");
}`

## performSelect方法

`[self performSelector:@selector(delayMethod) withObject:nil
 afterDelay:2.0];`

注：此方法是一种非阻塞的执行方式，未找到取消执行的方法,withObject可以传任何类型参数

<!-- more -->

## NSTimer方法

`NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:2.0 target:self selector:@selector(delayMethod) userInfo:nil repeats:NO];`

注：此方法是一种非阻塞的执行方式，取消执行方法：`- (void)invalidate;`即可

## NSThread进行现成sleep

`[NSThread sleepForTimeInterval:2.0];
[self delayMethod];
`

注：此方法是一种阻塞执行方式，建议放在子线程中执行，否则会卡住界面。没有找到取消执行方式。

## GCD方法

`__block ViewController *weakSelf = self;`

`dispatch_time_t delayTime = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC));`

`dispatch_after(delayTime, dispatch_get_main_queue(), ^{
    [weakSelf delayMethod];
});`


注：此方法是一种非阻塞执行方式。没有找到取消执行方式。可以在参数中选择执行的线程。


