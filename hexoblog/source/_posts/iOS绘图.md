---
title: iOS绘图
date: 2016-12-01 13:46:47
tags: [iOS,Core Graphics]
---

> Core Graphics Framework是一套基于C的API框架，使用了Quartz作为绘图引擎。它提供了低级别、轻量级、高保真度的2D渲染。该框架可以用于基于路径的绘图、变换、颜色管理、脱屏渲染，模板、渐变、遮蔽、图像数据管理、图像的创建、遮罩以及PDF文档的创建、显示和分析。

iOS支持两套图形API族：Core Graphics/QuartZ 2D 和OpenGL ES。OpenGL ES是跨平台的图形API，属于OpenGL的一个简化版本。QuartZ 2D是苹果公司开发的一套API，它是Core Graphics Framework的一部分。

<!-- more -->

Core Graphics API所有的操作都在一个上下文中进行。所以在绘图之前需要获取该上下文并传入执行渲染的函数中。如果你正在渲染一副在内存中的图片，此时就需要传入图片所属的上下文。获得一个图形上下文是我们完成绘图任务的第一步，你可以将图形上下文理解为一块画布。如果你没有得到这块画布，那么你就无法完成任何绘图操作。当然，有许多方式获得一个图形上下文，这里介绍两种最为常用的获取方法。


第一种方法就是创建一个图片类型的上下文。调用UIGraphicsBeginImageContextWithOptions函数就可获得用来处理图片的图形上下文。利用该上下文，你就可以在其上进行绘图，并生成图片。调用UIGraphicsGetImageFromCurrentImageContext函数可从当前上下文中获取一个UIImage对象。记住在你所有的绘图操作后别忘了调用UIGraphicsEndImageContext函数关闭图形上下文。

第二种方法是利用cocoa为你生成的图形上下文。当你子类化了一个UIView并实现了自己的drawRect：方法后，一旦drawRect：方法被调用，Cocoa就会为你创建一个图形上下文，此时你对图形上下文的所有绘图操作都会显示在UIView上。

判断一个上下文是否为当前图形上下文需要注意的几点：

UIGraphicsBeginImageContextWithOptions函数不仅仅是创建了一个适用于图形操作的上下文，并且该上下文也属于当前上下文。

当drawRect方法被调用时，UIView的绘图上下文属于当前图形上下文。

回调方法所持有的context：参数并不会让任何上下文成为当前图形上下文。此参数仅仅是对一个图形上下文的引用罢了。

## UIKit

像UIImage、NSString（绘制文本）、UIBezierPath（绘制形状）、UIColor都知道如何绘制自己。这些类提供了功能有限但使用方便的方法来让我们完成绘图任务。一般情况下，UIKit就是我们所需要的。

使用UiKit，你只能在当前上下文中绘图，所以如果你当前处于UIGraphicsBeginImageContextWithOptions函数或drawRect：方法中，你就可以直接使用UIKit提供的方法进行绘图。如果你持有一个context：参数，那么使用UIKit提供的方法之前，必须将该上下文参数转化为当前上下文。幸运的是，调用UIGraphicsPushContext 函数可以方便的将context：参数转化为当前上下文，记住最后别忘了调用UIGraphicsPopContext函数恢复上下文环境。

## Core Graphics

这是一个绘图专用的API族，它经常被称为QuartZ或QuartZ 2D。Core Graphics是iOS上所有绘图功能的基石，包括UIKit。

使用Core Graphics之前需要指定一个用于绘图的图形上下文（CGContextRef），这个图形上下文会在每个绘图函数中都会被用到。如果你持有一个图形上下文context：参数，那么你等同于有了一个图形上下文，这个上下文也许就是你需要用来绘图的那个。如果你当前处于UIGraphicsBeginImageContextWithOptions函数或drawRect：方法中，并没有引用一个上下文。为了使用Core Graphics，你可以调用UIGraphicsGetCurrentContext函数获得当前的图形上下文。

现在，我们有了两大绘图框架的支持以及三种获得图形上下文的方法（drawRect:、drawRect: inContext:、UIGraphicsBeginImageContextWithOptions）。那么我们就有6种绘图的形式。如果你有些困惑了，不用怕，我接下来将说明这6种情况。无需担心还没有具体的绘图命令，你只需关注上下文如何被创建以及我们是在使用UIKit还是Core Graphics。


## 绘图实例

* 在UIView的子类方法drawRect：中绘制一个红色圆，使用UIKit在Cocoa为我们提供的当前上下文中完成绘图任务。

```objective-c
 - (void) drawRect: (CGRect) rect {

UIBezierPath* p = [UIBezierPathbezierPathWithOvalInRect:CGRectMake(0,0,100,100)];

[[UIColor redColor] setFill];

[p fill];

}
 ```

* 使用Core Graphics实现绘制红色圆。

```objective-c
- (void) drawRect: (CGRect) rect {

CGContextRef con = UIGraphicsGetCurrentContext();

CGContextAddEllipseInRect(con, CGRectMake(0,0,100,100));

CGContextSetFillColorWithColor(con, [UIColor redColor].CGColor);

CGContextFillPath(con);

}
```
* 我将在UIView子类的`drawLayer:inContext：`方法中实现绘图任务。`drawLayer:inContext：`方法是一个绘制图层内容的代理方法。为了能够调用`drawLayer:inContext：`方法，我们需要设定图层的代理对象。但要注意，不应该将UIView对象设置为显示层的委托对象，这是因为UIView对象已经是隐式层的代理对象，再将它设置为另一个层的委托对象就会出问题。**轻量级的做法是：编写负责绘图形的代理类。**在MyTestView.h文件中声明如下代码：

```objective-c
@interface MyLayerDelegate : NSObject

@end
```
然后MyTestView.m文件中实现接口代码：

```objective-c
@implementation MyLayerDelegate

- (void)drawLayer:(CALayer*)layer inContext:(CGContextRef)ctx {

  UIGraphicsPushContext(ctx);

  UIBezierPath* p = [UIBezierPath bezierPathWithOvalInRect:CGRectMake(0,0,100,100)];

  [[UIColor redColor] setFill];

  [p fill];

  UIGraphicsPopContext();

}

@end
```

直接将代理类的实现代码放在MyTestView.m文件的#import代码的下面。**需要注意的是，我们所引用的上下文并不是当前上下文，所以为了能够使用UIKit，我们需要将引用的上下文转变成当前上下文。(UIGraphicsPushContext(ctx); )**

因为图层的代理是assign内存管理策略，那么这里就不能以局部变量的形式创建MyLayerDelegate实例对象赋值给图层代理。这里选择在MyTestView.m中增加一个实例变量，因为实例变量默认是strong。

```objective-c
@interface MyTestView () {

MyLayerDelegate* _layerDeleagete;

}

@end
```

使用该图层代理如下

```objective-c
MyTestView *myTestView = [[MyTestView alloc] initWithFrame: CGRectMake(0, 0, 320, 480)];

CALayer *myLayer = [CALayer layer];

_layerDelegate = [[MyLayerDelegate alloc] init];

myLayer.delegate = _layerDelegate;

[MyTestView.layer addSublayer:myLayer];

[MyTestView setNeedsDisplay]; // 调用此方法，drawLayer: inContext:方法才会被调用。
```
 * 使用Core Graphics在drawLayer:inContext：方法中实现同样操作，代码如下：

```objective-c
- (void)drawLayer:(CALayer*)lay inContext:(CGContextRef)con {

CGContextAddEllipseInRect(con, CGRectMake(0,0,100,100));

CGContextSetFillColorWithColor(con, [UIColor redColor].CGColor);

CGContextFillPath(con);

}
```

* 演示UIGraphicsBeginImageContextWithOptions的用法，并从上下文中生成一个UIImage对象。生成UIImage对象的代码并不需要等待某些方法被调用后或在UIView的子类中才能去做。

 UIGraphicsBeginImageContextWithOptions函数参数的含义：第一个参数表示所要创建的图片的尺寸；第二个参数用来指定所生成图片的背景是否为不透明，如上我们使用YES而不是NO，则我们得到的图片背景将会是黑色，显然这不是我想要的；第三个参数指定生成图片的缩放因子，这个缩放因子与UIImage的scale属性所指的含义是一致的。传入0则表示让图片的缩放因子根据屏幕的分辨率而变化，所以我们得到的图片不管是在单分辨率还是视网膜屏上看起来都会很好。

```objective-c
UIGraphicsBeginImageContextWithOptions(CGSizeMake(100,100), NO, 0);

UIBezierPath* p = [UIBezierPath bezierPathWithOvalInRect:CGRectMake(0,0,100,100)];

[[UIColor blueColor] setFill];

[p fill];

UIImage* im = UIGraphicsGetImageFromCurrentImageContext();

UIGraphicsEndImageContext();
```

 * 使用Core Graphics

```objective-c
UIGraphicsBeginImageContextWithOptions(CGSizeMake(100,100), NO, 0);

CGContextRef con = UIGraphicsGetCurrentContext();

CGContextAddEllipseInRect(con, CGRectMake(0,0,100,100));

CGContextSetFillColorWithColor(con, [UIColor blueColor].CGColor);

CGContextFillPath(con);

UIImage* im = UIGraphicsGetImageFromCurrentImageContext();

UIGraphicsEndImageContext();
```

**UIKit和Core Graphics可以在相同的图形上下文中混合使用。在iOS 4.0之前，使用UIKit和UIGraphicsGetCurrentContext被认为是线程不安全的。而在iOS4.0以后苹果让绘图操作在第二个线程中执行解决了此问题。**

# UImage 常用的绘图操作

一个UIImage对象提供了向当前上下文绘制自身的方法。我们现在已经知道如何获取一个图片类型的上下文并将它转变成当前上下文。

* 平移操作：下面的代码展示了如何将UIImage绘制在当前的上下文中。

```objective-c
UIImage* mars = [UIImage imageNamed:@"Mars.png"];

CGSize sz = [mars size];

UIGraphicsBeginImageContextWithOptions(CGSizeMake(sz.width*2, sz.height), NO, 0);

[mars drawAtPoint:CGPointMake(0,0)];

[mars drawAtPoint:CGPointMake(sz.width,0)];

UIImage* im = UIGraphicsGetImageFromCurrentImageContext();

UIGraphicsEndImageContext();

UIImageView* iv = [[UIImageView alloc] initWithImage:im];

[self.window.rootViewController.view addSubview: iv];

iv.center = self.window.center;
```

* 缩放操作：下面代码展示了如何对UIImage进行缩放操作：

```objective-c
UIImage* mars = [UIImage imageNamed:@"Mars.png"];

CGSize sz = [mars size];

UIGraphicsBeginImageContextWithOptions(CGSizeMake(sz.width*2, sz.height*2), NO, 0);

[mars drawInRect:CGRectMake(0,0,sz.width*2,sz.height*2)];

[mars drawInRect:CGRectMake(sz.width/2.0, sz.height/2.0, sz.width, sz.height) blendMode:kCGBlendModeMultiply alpha:1.0];

UIImage* im = UIGraphicsGetImageFromCurrentImageContext();

UIGraphicsEndImageContext();
```

* 裁剪操作：下面代码展示了如何获取图片的右半边：

 UIImage没有提供截取图片指定区域的功能。但通过创建一个较小的图形上下文并移动图片到一个适当的图形上下文坐标系内，指定区域内的图片就会被获取。

```objective-c
UIImage* mars = [UIImage imageNamed:@"Mars.png"];

CGSize sz = [mars size];

UIGraphicsBeginImageContextWithOptions(CGSizeMake(sz.width/2.0, sz.height), NO, 0);

[mars drawAtPoint:CGPointMake(-sz.width/2.0, 0)];

UIImage* im = UIGraphicsGetImageFromCurrentImageContext();

UIGraphicsEndImageContext();
```
以上的代码首先创建一个一半图片宽度的图形上下文，然后将图片左上角原点移动到与图形上下文负X坐标对齐，从而让图片只有右半部分与图形上下文相交。

# CGImage常用的绘图操作

UIImage的Core Graphics版本是CGImage（具体类型是CGImageRef）。两者可以直接相互转化: 使用UIImage的CGImage属性可以访问Quartz图片数据；将CGImage作为UIImage方法`imageWithCGImage:`或`initWithCGImage:`的参数创建UIImage对象。

一个CGImage对象可以让你获取原始图片中指定区域的图片（也可以获取指定区域外的图片，UIImage却办不到）。

下面的代码展示了将图片拆分成两半，并分别绘制在上下文的左右两边：

```objective-c
UIImage* mars = [UIImage imageNamed:@"Mars.png"];

// 抽取图片的左右半边

CGSize sz = [mars size];

CGImageRef marsLeft = CGImageCreateWithImageInRect([mars CGImage],CGRectMake(0,0,sz.width/2.0,sz.height));

CGImageRef marsRight = CGImageCreateWithImageInRect([mars CGImage],CGRectMake(sz.width/2.0,0,sz.width/2.0,sz.height));

// 将每一个CGImage绘制到图形上下文中

UIGraphicsBeginImageContextWithOptions(CGSizeMake(sz.width*1.5, sz.height), NO, 0);

CGContextRef con = UIGraphicsGetCurrentContext();

CGContextDrawImage(con, CGRectMake(0,0,sz.width/2.0,sz.height), marsLeft);

CGContextDrawImage(con, CGRectMake(sz.width,0,sz.width/2.0,sz.height), marsRight);

UIImage* im = UIGraphicsGetImageFromCurrentImageContext();

UIGraphicsEndImageContext();

// 记得释放内存，ARC在这里无效

CGImageRelease(marsLeft);

CGImageRelease(marsRight);
```

**你也许发现绘出的图是上下颠倒的！**

图片的颠倒并不是因为被旋转了。当你创建了一个CGImage并使用CGContextDrawImage方法绘图就会引起这种问题。这主要是因为原始的本地坐标系统（坐标原点在左上角）与目标上下文（坐标原点在左下角）不匹配。

有很多方法可以修复这个问题，其中一种方法就是使用CGContextDrawImage方法先将CGImage绘制到UIImage上，然后获取UIImage对应的CGImage，此时就得到了一个倒转的CGImage。当再调用CGContextDrawImage方法，我们就将倒转的图片还原回来了。实现代码如下：

```objective-c
CGImageRef flip (CGImageRef im) {

CGSize sz = CGSizeMake(CGImageGetWidth(im), CGImageGetHeight(im));

UIGraphicsBeginImageContextWithOptions(sz, NO, 0);

CGContextDrawImage(UIGraphicsGetCurrentContext(), CGRectMake(0, 0, sz.width, sz.height), im);

CGImageRef result = [UIGraphicsGetImageFromCurrentImageContext() CGImage];

UIGraphicsEndImageContext();

return result;

}
```

之前代码修改如下：

```objective-c
CGContextDrawImage(con, CGRectMake(0,0,sz.width/2.0,sz.height), flip(marsLeft));

CGContextDrawImage(con, CGRectMake(sz.width,0,sz.width/2.0,sz.height), flip(marsRight));
```
**在双分辨率的设备上，我们的图片文件是高分辨率（@2x）版本，上面的绘图就是错误的。**

原因在于对于UIImage来说，在加载原始图片时使用`imageNamed:`方法，它会自动根据所在设备的分辨率类型选择图片，并且UIImage通过设置用来适配的scale属性补偿图片的两倍尺寸。但是一个CGImage对象并没有scale属性，它不知道图片文件的尺寸是否为两倍！所以当调用UIImage的CGImage方法，你不能假定所获得的CGImage尺寸与原始UIImage是一样的。在单分辨率和双分辨率下，一个UIImage对象的size属性值都是一样的，但是双分辨率UIImage对应的CGImage是单分辨率UIImage对应的CGImage的两倍大。所以我们需要修改上面的代码，让其在单双分辨率下都可以工作。代码如下：

```objective-c
UIImage* mars = [UIImage imageNamed:@"Mars.png"];

CGSize sz = [mars size];

// 转换CGImage并使用对应的CGImage尺寸截取图片的左右部分

CGImageRef marsCG = [mars CGImage];

CGSize szCG = CGSizeMake(CGImageGetWidth(marsCG), CGImageGetHeight(marsCG));

CGImageRef marsLeft = CGImageCreateWithImageInRect(marsCG,CGRectMake(0,0,szCG.width/2.0,szCG.height));

CGImageRef marsRight = CGImageCreateWithImageInRect(marsCG, CGRectMake(szCG.width/2.0,0,szCG.width/2.0,szCG.height));

UIGraphicsBeginImageContextWithOptions(CGSizeMake(sz.width*1.5, sz.height), NO, 0);

//剩下的和之前的代码一样，修复倒置问题

CGContextRef con = UIGraphicsGetCurrentContext();

CGContextDrawImage(con, CGRectMake(0,0,sz.width/2.0,sz.height),flip(marsLeft));

CGContextDrawImage(con, CGRectMake(sz.width,0,sz.width/2.0,sz.height),flip(marsRight));

UIImage* im = UIGraphicsGetImageFromCurrentImageContext();

UIGraphicsEndImageContext();

CGImageRelease(marsLeft);

CGImageRelease(marsRight);

```
上面的代码初看上去很繁杂，不过不用担心，这里还有另一种修复倒置问题的方案。相对于使用flip函数，你可以在绘图之前将CGImage包装进UIImage中，这样做有两大优点：
1.当UIImage绘图时它会自动修复倒置问题
2.当你从CGImage转化为Uimage时，可调用`imageWithCGImage:scale:orientation:`方法生成CGImage作为对缩放性的补偿。

所以这是一个解决倒置和缩放问题的自包含方法。

 ```objective-c
 UIImage* mars = [UIImage imageNamed:@"Mars.png"];

CGSize sz = [mars size];

CGImageRef marsCG = [mars CGImage];

CGSize szCG = CGSizeMake(CGImageGetWidth(marsCG), CGImageGetHeight(marsCG));

CGImageRef marsLeft = CGImageCreateWithImageInRect(marsCG, CGRectMake(0,0,szCG.width/2.0,szCG.height));

CGImageRef marsRight = CGImageCreateWithImageInRect(marsCG, CGRectMake(szCG.width/2.0,0,szCG.width/2.0,szCG.height));

UIGraphicsBeginImageContextWithOptions(CGSizeMake(sz.width*1.5, sz.height), NO, 0);

[[UIImage imageWithCGImage:marsLeft scale:[mars scale] orientation:UIImageOrientationUp] drawAtPoint:CGPointMake(0,0)];

[[UIImage imageWithCGImage:marsRight scale:[mars scale] orientation:UIImageOrientationUp] drawAtPoint:CGPointMake(sz.width,0)];

UIImage* im = UIGraphicsGetImageFromCurrentImageContext();

UIGraphicsEndImageContext();

CGImageRelease(marsLeft); CGImageRelease(marsRight);
 ```
 还有另一种解决倒置问题的方案是在绘制CGImage之前，对上下文应用变换操作，有效地倒置上下文的内部坐标系统。这里先不做讨论。


参考文档

1. [iOS绘图教程](http://blog.csdn.net/huangznian/article/details/42741039)

