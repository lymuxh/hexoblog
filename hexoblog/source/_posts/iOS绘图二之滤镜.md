---
title: iOS绘图二之滤镜
date: 2016-12-01 16:49:48
tags: [绘图,CIFilter,CIImage]
---

# iOS滤镜

CIFilter与CIImage是iOS 5新引入的，虽然它们已在MAX OS X系统中存在多年。前缀“CI”表示Core Image，这是一种使用数学滤镜变换图片的技术。但是你不要去幻想iOS提供了像Photoshop软件那样强大的滤镜功能。使用Core Image之前你需要将CoreImage.framework框架导入到你的target之中。

<!-- more -->

## CIFilter

所谓滤镜指的是CIFilter类，滤镜可被分为以下几类：

**模板与渐变类**

这两类滤镜创建的CIImage可以和其他的CIImage进行合并，比如一种单色，一个棋盘，条纹，亦或是渐变。

**合成类**

此类滤镜可以将一张图片与另外的图片合并，合成滤镜模式常见于图形处理软件Photoshop中。

**色彩类**

此滤镜调整、修改图片的色彩。因此你可以改变一张图片的饱和度、色度、亮度、对比度、伽马、白点、曝光度、阴影、高亮等属性。

**几何变换类**

此类滤镜可对图片执行基本的几何变换，比如缩放、旋转、裁剪。

CIFilter使用起来非常的简单。CIFilter看上去就像一个由键值组成的字典。它生成一个CIImage对象作为其输出。一般地，一个滤镜有一个或多个输入，而对于部分滤镜，生成的图片是基于其他类型的参数值。

CIFilter对象是一个集合，可使用键值对进行检索。通过提供滤镜的字符串名称创建一个滤镜，如果想知道有哪些滤镜，可以查询苹果的**Core Image Filter Reference**文档，或是调用CIFilter的类方法`filterNamesInCategories：`，参数值为nil。每一个滤镜拥有一小部分用来确定其行为的键值。如果你想修改某一个键（比如亮度键）对应的值，你可以调用`setValue：forKey：`方法或当你指定一个滤镜名时提供所有键值对。

需要处理的图片必须是CIImage类型，调用`initWithCGImage：`方法可获得CIImage。因为CGImage又是作为滤镜的输出，因此滤镜之间可被连接在一起（将滤镜的输出作为`initWithCGImage：`方法的输入参数）

当你构建一个滤镜链时，并没有做复杂的运算。只有当整个滤镜链需要输出一个CGImage时，密集型计算才会发生。调用`contextWithOptions：`和`createCGImage: fromRect:`方法创建CIContext。与以往不同的地方是CIImage没有frame与bounds属性；只有extent属性。你将非常频繁的使用这个属性作为`createCGImage: fromRect:`方法的第二个参数。

接下来我将演示Core Image的使用。首先创建一个径向渐变的滤镜，该滤镜是从白到黑的渐变方式，白色区域的半径默认是100。接着将其与一张使用CIDarkenBlendMode滤镜的图片合成。CIDarkenBlendMode的作用是背景图片样本将被源图片的黑色部分替换掉。

```objective-c
UIImage* moi = [UIImage imageNamed:@"Mars.jpeg"];

CIImage* moi2 = [[CIImage alloc] initWithCGImage:moi.CGImage];

CIFilter* grad = [CIFilter filterWithName:@"CIRadialGradient"];

CIVector* center = [CIVector vectorWithX:moi.size.width / 2.0 Y:moi.size.height / 2.0];

// 使用setValue：forKey：方法设置滤镜属性

[grad setValue:center forKey:@"inputCenter"];

// 在指定滤镜名时提供所有滤镜键值对

CIFilter* dark = [CIFilter filterWithName:@"CIDarkenBlendMode" keysAndValues:@"inputImage", grad.outputImage, @"inputBackgroundImage", moi2, nil];

CIContext* c = [CIContext contextWithOptions:nil];

CGImageRef moi3 = [c createCGImage:dark.outputImage fromRect:moi2.extent];

UIImage* moi4 = [UIImage imageWithCGImage:moi3 scale:moi.scale orientation:moi.imageOrientation];

CGImageRelease(moi3);
```

Core Image是使用GPU处理。Core Graphics也可以做到径向渐变并使用混合模式合成图片。但Core Image要简单得多，特别是当你有多个图片输入想重用一个滤镜链时。并且Core Image的颜色调整功能比Core Graphics更加强大。对了，Core Image还能实现自动人脸识别哦！

## 绘制一个UIView

绘制一个UIVIew最灵活的方式就是由它自己完成绘制。实际上你不是绘制一个UIView，你只是子类化了UIView并赋予子类绘制自己的能力。当一个UIVIew需要执行绘图操作的时，drawRect:方法就会被调用。

覆盖此方法让你获得绘图操作的机会。当drawRect：方法被调用，当前图形上下文也被设置为属于视图的图形上下文。你可以使用Core Graphics或UIKit提供的方法将图形画到该上下文中。

你不应该手动调用drawRect：方法！如果你想调用drawRect：方法更新视图，只需发送setNeedsDisplay方法。这将使得drawRect：方法会在下一个适当的时间调用。

当然，不要覆盖drawRect：方法除非你知道这样做绝对合法。比方说，在UIImageView子类中覆盖drawRect：方法是不合法的，你将得不到你绘制的图形。

在UIView子类的drawRect：方法中无需调用super，因为本身UIView的drawRect：方法是空的。为了提高一些绘图性能，你可以调用setNeedsDisplayInRect方法重新绘制视图的子区域，而视图的其他部分依然保持不变。

一般情况下，你不应该过早的进行优化。绘图代码可能看上去非常的繁琐，但它们是非常快的。并且iOS绘图系统自身也是非常高效，它不会频繁调用drawRect：方法，除非迫不得已（或调用了setNeedsDisplay方法）。

一旦一个视图已由自己绘制完成，那么绘制的结果会被缓存下来留待重用，而不是每次重头再来。(苹果公司将缓存绘图称为视图的位图存储回填（bitmap backing store）)。你可能会发现drawRect：方法中的代码在整个应用程序生命周期内只被调用了一次！事实上，将代码移到drawRect：方法中是提高性能的普遍做法。这是因为绘图引擎直接对屏幕进行渲染相对于先是脱屏渲染然后再将像素拷贝到屏幕要来的高效。

当视图的backgroundColor为nil并且opaque属性为YES，视图的背景颜色就会变成黑色。

## Core Graphics上下文属性设置

当你在图形上下文中绘图时，当前图形上下文的相关属性设置将决定绘图的行为与外观。因此，绘图的一般过程是先设定好图形上下文参数，然后绘图。比方说，要画一根红线，接着画一根蓝线。那么首先需要将上下文的线条颜色属性设定为为红色，然后画红线；接着设置上下文的线条颜色属性为蓝色，再画出蓝线。表面上看,红线和蓝线是分开的，但事实上，在你画每一条线时，线条颜色却是整个上下文的属性。无论你用的是UIKit方法还是Core Graphics函数。

因为图形上下文在每一时刻都有一个确定的状态，该状态概括了图形上下文所有属性的设置。为了便于操作这些状态，图形上下文提供了一个用来持有状态的栈。调用CGContextSaveGState函数，上下文会将完整的当前状态压入栈顶；调用CGContextRestoreGState函数，上下文查找处在栈顶的状态，并设置当前上下文状态为栈顶状态。

因此一般绘图模式是：在绘图之前调用`CGContextSaveGState`函数保存当前状态，接着根据需要设置某些上下文状态，然后绘图，最后调用`CGContextRestoreGState`函数将当前状态恢复到绘图之前的状态。要注意的是，`CGContextSaveGState`函数和`CGContextRestoreGState`函数必须成对出现，否则绘图很可能出现意想不到的错误，这里有一个简单的做法避免这种情况。代码如下：

```objective-c
- (void)drawRect:(CGRect)rect {

CGContextRef ctx = UIGraphicsGetCurrentContext();

CGContextSaveGState(ctx);

{

// 绘图代码

}

CGContextRestoreGState(ctx);

} 　
```
但你不需要在每次修改上下文状态之前都这样做，因为你对某一上下文属性的设置并不一定会和之前的属性设置或其他的属性设置产生冲突。你完全可以在不调用保存和恢复函数的情况下先设置线条颜色为红色，然后再设置为蓝色。但在一定情况下，你希望你对状态的设置是可撤销的，我将在接下来讨论这样的情况。

许多的属性组成了一个图形上下文状态，这些属性设置决定了在你绘图时图形的外观和行为。下面我列出了一些属性和对应修改属性的函数；虽然这些函数是关于Core Graphics的，但记住，实际上UIKit同样是调用这些函数操纵上下文状态。

**线条的宽度和线条的虚线样式**

CGContextSetLineWidth、CGContextSetLineDash

**线帽和线条联接点样式**

CGContextSetLineCap、CGContextSetLineJoin、CGContextSetMiterLimit

**线条颜色和线条模式**

CGContextSetRGBStrokeColor、CGContextSetGrayStrokeColor、CGContextSetStrokeColorWithColor、CGContextSetStrokePattern

**填充颜色和模式**

CGContextSetRGBFillColor,CGContextSetGrayFillColor,CGContextSetFillColorWithColor, CGContextSetFillPattern

**阴影**

CGContextSetShadow、CGContextSetShadowWithColor

**混合模式**

CGContextSetBlendMode（决定你当前绘制的图形与已经存在的图形如何被合成）

**整体透明度**

CGContextSetAlpha（个别颜色也具有alpha成分）

**文本属性**

CGContextSelectFont、CGContextSetFont、CGContextSetFontSize、CGContextSetTextDrawingMode、CGContextSetCharacterSpacing

**是否开启反锯齿和字体平滑**

CGContextSetShouldAntialias、CGContextSetShouldSmoothFonts


裁剪区域:在裁剪区域外绘图不会被实际的画出来。

变换（或称为“CTM“，意为当前变换矩阵): 改变你随后指定的绘图命令中的点如何被映射到画布的物理空间。

## 路径与绘图

通过编写移动虚拟画笔的代码描画一段路径，这样的路径并不构成一个图形。绘制路径意味着对路径描边或填充该路径，也或者两者都做。同样，你应该从某些绘图程序中得到过相似的体会。

一段路径是由点到点的描画构成。想象一下绘图系统是你手里的一只画笔，你首先必须要设置画笔当前所处的位置，然后给出一系列命令告诉画笔如何描画随后的每段路径。每一段新增的路径开始于当前点，当完成一条路径的描画，路径的终点就变成了当前点。

下面列出了一些路径描画的命令：

**定位当前点**

CGContextMoveToPoint

**描画一条线**

CGContextAddLineToPoint、CGContextAddLines

**描画一个矩形**

CGContextAddRect、CGContextAddRects

**描画一个椭圆或圆形**

CGContextAddEllipseInRect

**描画一段圆弧**

CGContextAddArcToPoint、CGContextAddArc

**通过一到两个控制点描画一段贝赛尔曲线**

CGContextAddQuadCurveToPoint、CGContextAddCurveToPoint

**关闭当前路径**

CGContextClosePath

这将从路径的终点到起点追加一条线。如果你打算填充一段路径，那么就不需要使用该命令，因为该命令会被自动调用。

**描边或填充当前路径**

CGContextStrokePath、CGContextFillPath、CGContextEOFillPath、CGContextDrawPath。

对当前路径描边或填充会清除掉路径。如果你只想使用一条命令完成描边和填充任务，可以使用CGContextDrawPath命令，因为如果你只是使用CGContextStrokePath对路径描边，路径就会被清除掉，你就不能再对它进行填充了。

创建路径并描边路径或填充路径只需一条命令就可完成的函数：CGContextStrokeLineSegments、CGContextStrokeRect、CGContextStrokeRectWithWidth、CGContextFillRect、CGContextFillRects、CGContextStrokeEllipseInRect、CGContextFillEllipseInRect。

一段路径是被合成的，意思是它是由多条独立的路径组成。举个例子，一条单独的路径可能由两个独立的闭合形状组成：一个矩形和一个圆形。当你在构造一条路径的中间过程（意思是在描画了一条路径后没有调用描边或填充命令，或调用CGContextBeginPath函数来清除路径）调用CGContextMoveToPoint函数，就像是你拾起画笔，并将画笔移动到一个新的位置，如此来准备开始一段独立的相同路径。如果你担心当你开始描画一条路径的时候，已经存在的路径和新的路径会被认为是已存在路径的一个合成部分，你可以调用CGContextBeginPath函数指定你绘制的路径是一条独立的路径；苹果的许多例子都是这样做的，但在实际开发中我发现这是非必要的。

CGContextClearRect函数的功能是擦除一个区域。这个函数会擦除一个矩形内的所有已存在的绘图；并对该区域执行裁剪。结果像是打了一个贯穿所有已存在绘图的孔。

CGContextClearRect函数的行为依赖于上下文是透明还是不透明。当在图形上下文中绘图时，这会尤为明显和直观。如果图片上下文是透明的（UIGraphicsBeginImageContextWithOptions第二个参数为NO），那么CGContextClearRect函数执行擦除后的颜色为透明，反之则为黑色。

当在一个视图中直接绘图（使用drawRect：或drawLayer：inContext：方法），如果视图的背景颜色为nil或颜色哪怕有一点点透明度，那么CGContextClearRect的矩形区域将会显示为透明的，打出的孔将穿过视图包括它的背景颜色。如果背景颜色完全不透明，那么CGContextClearRect函数的结果将会是黑色。这是因为视图的背景颜色决定了是否视图的图形上下文是透明的还是不透明的。

但是这却完全改变了CGContextClearRect函数的效果。UIView子类的drawRect：方法看起来像这样：

```objective-c
CGContextRef con = UIGraphicsGetCurrentContext();

CGContextSetFillColorWithColor(con, [UIColor blueColor].CGColor);

CGContextFillRect(con, rect);

CGContextClearRect(con, CGRectMake(0,0,30,30));
```

我们应该在绘图代码周围使用`CGContextSaveGState`和`CGContextRestoreGState`函数。上下文在调用`drawRect：`方法中不会被持久，所以不会被破坏。

如果一段路径需要重用或共享，你可以将路径封装为CGPath（具体类型是CGPathRef）。你可以创建一个新的CGMutablePathRef对象并使用多个类似于图形的路径函数的CGPath函数构造路径，或者使用CGContextCopyPath函数复制图形上下文的当前路径。有许多CGPath函数可用于创建基于简单几何形状的路径（CGPathCreateWithRect、CGPathCreateWithEllipseInRect）或基于已存在路径（CGPathCreateCopyByStrokingPath、CGPathCreateCopyDashingPath、CGPathCreateCopyByTransformingPath）。

UIKit的UIBezierPath类包装了CGPath。它提供了用于绘制某种形状路径的方法，以及用于描边、填充、存取某些当前上下文状态的设置方法。类似地，UIColor提供了用于设置当前上下文描边与填充的颜色。

```objective-c
UIBezierPath* p = [UIBezierPath bezierPath];

[p moveToPoint:CGPointMake(100,100)];

[p addLineToPoint:CGPointMake(100, 19)];

[p setLineWidth:20];

[p stroke];

[[UIColor redColor] set];

[p removeAllPoints];

[p moveToPoint:CGPointMake(80,25)];

[p addLineToPoint:CGPointMake(100, 0)];

[p addLineToPoint:CGPointMake(120, 25)];

[p fill];

[p removeAllPoints];

[p moveToPoint:CGPointMake(90,101)];

[p addLineToPoint:CGPointMake(100, 90)];

[p addLineToPoint:CGPointMake(110, 101)];

[p fillWithBlendMode:kCGBlendModeClear alpha:1.0];

```

## 圆角

如果你需要对象特性，UIBezierPath提供了一个便利方法：`bezierPathWithRoundedRect：cornerRadius：`，它可用于绘制带有圆角的矩形，如果是使用Core Graphics就相当冗长乏味了。还可以只让圆角出现在左上角和右上角。

```objective-c
- (void)drawRect:(CGRect)rect {

  CGContextRef ctx = UIGraphicsGetCurrentContext();

  CGContextSetStrokeColorWithColor(ctx, [UIColor blackColor].CGColor);

  CGContextSetLineWidth(ctx, 3);

  UIBezierPath *path;

  path = [UIBezierPath bezierPathWithRoundedRect:CGRectMake(100, 100, 100, 100) byRoundingCorners:(UIRectCornerTopLeft |UIRectCornerTopRight) cornerRadii:CGSizeMake(10, 10)];

  [path stroke];

}
```

## 裁剪

路径的另一用处是遮蔽区域，以防对遮蔽区域进一步绘图。这种用法被称为裁剪。裁剪区域外的图形不会被绘制到。默认情况下，一个图形上下文的裁剪区域是整个图形上下文。你可在上下文中的任何地方绘图。

总的来说，裁剪区域是上下文的一个特性。与已存在的裁剪区域相交会出现新的裁剪区域。所以如果你应用了你自己的裁剪区域，稍后将它从图形上下文中移除的做法是使用CGContextSaveGState和CGContextRestoreGState函数将代码包装起来。

为了便于说明这一点，我使用裁剪而不是使用混合模式在箭头杆子上打孔的方法重写了生成箭头的代码。这样做有点小复杂，因为我们想要裁剪区域不在三角形内而在三角形外部。为了表明这一点，我们使用了一个三角形和一个矩形组成了一个组合路径。

当填充一个组合路径并使用它表示一个裁剪区域时，系统遵循以下两规则之一：

**环绕规则（Winding rule）**

如果边界是顺时针绘制，那么在其内部逆时针绘制的边界所包含的内容为空。如果边界是逆时针绘制，那么在其内部顺时针绘制的边界所包含的内容为空。

**奇偶规则**

最外层的边界代表内部都有效，都要填充；之后向内第二个边界代表它的内部无效，不需填充；如此规则继续向内寻找边界线。我们的情况非常简单，所以使用奇偶规则就很容易了。这里我们使用CGContextEOCllip设置裁剪区域然后进行绘图。

```objective-c
CGContextRef con = UIGraphicsGetCurrentContext();

// 在上下文裁剪区域中挖一个三角形状的孔

CGContextMoveToPoint(con, 90, 100);

CGContextAddLineToPoint(con, 100, 90);

CGContextAddLineToPoint(con, 110, 100);

CGContextClosePath(con);

CGContextAddRect(con, CGContextGetClipBoundingBox(con));

// 使用奇偶规则，裁剪区域为矩形减去三角形区域

CGContextEOClip(con);

// 绘制垂线

CGContextMoveToPoint(con, 100, 100);

CGContextAddLineToPoint(con, 100, 19);

CGContextSetLineWidth(con, 20);

CGContextStrokePath(con);

// 画红色箭头

CGContextSetFillColorWithColor(con, [[UIColor redColor] CGColor]);

CGContextMoveToPoint(con, 80, 25);

CGContextAddLineToPoint(con, 100, 0);

CGContextAddLineToPoint(con, 120, 25);

CGContextFillPath(con);
```

## 渐变

渐变可以很简单也可以很复杂。一个简单的渐变（接下来要讨论的）由一端点的颜色与另一端点的颜色决定，如果在中间点加入颜色（可选），那么渐变会在上下文的两个点之间线性的绘制或在上下文的两个圆之间放射状的绘制。不能使用渐变作为路径的填充色，但可使用裁剪限制对路径形状的渐变。

```objective-c
// 绘制渐变

CGFloat locs[3] = { 0.0, 0.5, 1.0 };

CGFloat colors[12] = {

0.3,0.3,0.3,0.8, // 开始颜色，透明灰

0.0,0.0,0.0,1.0, // 中间颜色，黑色

0.3,0.3,0.3,0.8 // 末尾颜色，透明灰

};

CGColorSpaceRef sp = CGColorSpaceCreateDeviceGray();

CGGradientRef grad = CGGradientCreateWithColorComponents (sp, colors, locs, 3);

CGContextDrawLinearGradient(con, grad, CGPointMake(89,0), CGPointMake(111,0), 0);

CGColorSpaceRelease(sp);

CGGradientRelease(grad);

CGContextRestoreGState(con); // 完成裁剪

```

调用CGContextReplacePathWithStrokedPath函数假装对当前路径描边，并使用当前线段宽度和与线段相关的上下文状态设置。但接着创建的是描边路径外部的一个新的路径。因此，相对于使用粗的线条，我们使用了一个矩形区域作为裁剪区域。

虽然过程比较冗长但是非常的简单；我们将渐变描述为一组在一端点（0.0）和另一端点（1.0）之间连续区上的位置，以及设置与每个位置相对应的颜色。为了提亮边缘的渐变，加深中间的渐变，我使用了三个位置，黑色点的位置是0.5。为了创建渐变，还需要提供一个颜色空间。最后，我创建出了该渐变，并对裁剪区域绘制线性渐变，最后释放了颜色空间和渐变。


## 颜色与模版

在iOS中，CGColor表示颜色（具体类型为CGColorRef）。使用UIColor的colorWithCGColor：和CGColor方法可bridged cast到UIColor。

在iOS中，模板表示为CGPattern（具体类型为CGPatternRef）。你可以创建一个模板并使用它进行描边或填充。其过程是相当复杂的。作为一个非常简单的例子，我将使用红蓝相间的三角形替换箭头的三角形部分。现在移除下面行：

CGContextSetFillColorWithColor（con， [UIColor redColor].CGColor））；

在被移除的地方填入下面代码：

```objective-c
CGColorSpaceRef sp2 = CGColorSpaceCreatePattern(NULL);

CGContextSetFillColorSpace (con, sp2);

CGColorSpaceRelease (sp2);

CGPatternCallbacks callback = {0, &drawStripes, NULL };

CGAffineTransform tr = CGAffineTransformIdentity;

CGPatternRef patt = CGPatternCreate(NULL,CGRectMake(0,0,4,4), tr, 4, 4, kCGPatternTilingConstantSpacingMinimalDistortion, true, &callback);

CGFloat alph = 1.0;

CGContextSetFillPattern(con, patt, &alph);

CGPatternRelease(patt);
```
代码非常冗长，但它却是一个完整的样板。现在我们从后往前分析代码: 我们调用CGContextSetFillPattern不是设置填充颜色，我们设置的是填充的模板。函数的第三个参数是一个指向CGFloat的指针，所以我们事先设置CGFloat自身。第二个参数是一个CGPatternRef对象，所以我们需要事先创建CGPatternRef，并在最后释放它。


使用颜色模板调用CGContextSetFillPattern函数之前，你需要设置将应用到模板颜色空间的上下文填充颜色空间。如果你忽略这项工作，那么当你调用CGContextSetFillPattern函数时会发生错误。所以我们创建了颜色空间，设置它作为上下文的填充颜色空间，并在后面做了释放。

到这里我们仍然没有完成绘图。因为我还没有编写向矩形元中绘图的函数！绘图函数地址被表示为&drawStripes。绘图代码如下所示：

```objective-c
void drawStripes (void *info, CGContextRef con) {

// assume 4 x 4 cell

CGContextSetFillColorWithColor(con, [[UIColor redColor] CGColor]);

CGContextFillRect(con, CGRectMake(0,0,4,4));

CGContextSetFillColorWithColor(con, [[UIColor blueColor] CGColor]);

CGContextFillRect(con, CGRectMake(0,0,4,2));

}
```


## 图形上下文变换

就像UIView可以实现变换，同样图形上下文也具备这项功能。然而对图形上下文应用一个变换操作不会对已在图形上下文上的绘图产生什么影响，它只会影响到在上下文变换之后被绘制的图形，并改变被映射到图形上下文区域的坐标方式。一个图形上下文变换被称为CTM，意为“当前变换矩阵“（current transformation matrix）。

完全利用图形上下文的CTM来免于即使是简单的计算操作是很常见的。你可以使用CGContextConcatCTM函数将当前变换乘上任何CGAffineTransform，还有一些便利函数可对当前变换应用平移、缩放，旋转变换。

当你获得上下文的时候，对图形上下文的基本变换已经设置好了；这就是系统能映射上下文绘图坐标到屏幕坐标的原因。无论你对当前变换应用了什么变换，基本变换变换依然有效并且绘图继续工作。通过将你的变换代码封装到CGContextSaveGState和CGContextRestoreGState函数调用中，对基本变换应用的变换操作可以被还原。

举个例子，对于我们迄今为止使用代码绘制的向上箭头来说，已知的放置箭头的方式仅仅只有一个位置：箭头矩形框的左上角被硬编码在坐标{80，0}。这样代码很难理解、灵活性差、且很难被重用。最明智的做法是通过将所有代码中的x坐标值减去80，让箭头矩形框左上角在坐标{0，0}。事先应用一个简单的平移变换，很容易将箭头画在任何位置。为了映射坐标到箭头的左上角，我们使用下面代码：

`CGContextTranslateCTM（con, 80, 0）; `//在坐标{0,0}处绘制箭头

旋转变换特别的有用，它可以让你在一个被旋转的方向上进行绘制而无需使用任何复杂的三角函数。然而这略有点复杂，因为旋转变换围绕的点是原点坐标。这几乎不是你所想要的，所以你先是应用了一个平移变换，为的是映射原点到你真正想绕其旋转的点。但是接着，在旋转之后，为了算出你在哪里绘图，你可能需要做一次逆向平移变换。
为了说明这个做法，我将绕箭头杆子尾部旋转多个角度重复绘制箭头，并把对箭头的绘图封装为UIImage对象。接着我们简单重复绘制UIImage对象。

```objective-c
- (void)drawRect:(CGRect)rect {

UIGraphicsBeginImageContextWithOptions(CGSizeMake(40,100), NO, 0.0);

CGContextRef con = UIGraphicsGetCurrentContext();

CGContextSaveGState(con);

CGContextMoveToPoint(con, 90 - 80, 100);

CGContextAddLineToPoint(con, 100 - 80, 90);

CGContextAddLineToPoint(con, 110 - 80, 100);

CGContextMoveToPoint(con, 110 - 80, 100);

CGContextAddLineToPoint(con, 100 - 80, 90);

CGContextAddLineToPoint(con, 90 - 80, 100);

CGContextClosePath(con);

CGContextAddRect(con, CGContextGetClipBoundingBox(con));

CGContextEOClip(con);

CGContextMoveToPoint(con, 100 - 80, 100);

CGContextAddLineToPoint(con, 100 - 80, 19);

CGContextSetLineWidth(con, 20);

CGContextReplacePathWithStrokedPath(con);

CGContextClip(con);

CGFloat locs[3] = { 0.0, 0.5, 1.0 };

CGFloat colors[12] = {

0.3,0.3,0.3,0.8,

0.0,0.0,0.0,1.0,

0.3,0.3,0.3,0.8

};

CGColorSpaceRef sp = CGColorSpaceCreateDeviceGray();

CGGradientRef grad = CGGradientCreateWithColorComponents (sp, colors, locs, 3);

CGContextDrawLinearGradient (con, grad, CGPointMake(89 - 80,0), CGPointMake(111 - 80,0), 0);

CGColorSpaceRelease(sp);

CGGradientRelease(grad);

CGContextRestoreGState(con);

CGColorSpaceRef sp2 = CGColorSpaceCreatePattern(NULL);

CGContextSetFillColorSpace (con, sp2);

CGColorSpaceRelease (sp2);

CGPatternCallbacks callback = {0, &drawStripes, NULL };

CGAffineTransform tr = CGAffineTransformIdentity;

CGPatternRef patt = CGPatternCreate(NULL,CGRectMake(0,0,4,4),tr,4,4，kCGPatternTilingConstantSpacingMinimalDistortion,true, &callback);

CGFloat alph = 1.0;

CGContextSetFillPattern(con, patt, &alph);

CGPatternRelease(patt);

CGContextMoveToPoint(con, 80 - 80, 25);

CGContextAddLineToPoint(con, 100 - 80, 0);

CGContextAddLineToPoint(con, 120 - 80, 25);

CGContextFillPath(con);

UIImage* im = UIGraphicsGetImageFromCurrentImageContext();

UIGraphicsEndImageContext();

con = UIGraphicsGetCurrentContext();

[im drawAtPoint:CGPointMake(0,0)];

for (int i=0; i<3; i++) {

CGContextTranslateCTM(con, 20, 100);

CGContextRotateCTM(con, 30 * M_PI/180.0);

CGContextTranslateCTM(con, -20, -100);

[im drawAtPoint:CGPointMake(0,0)];

}

}
```
变换有多个方法解决我们早期使用CGContextDrawImage函数遇到的倒置问题。相对于逆向绘图，我们选择逆向我们绘图的上下文。实质上，我们对上下文坐标系统应用了一个“倒置”变换。你自上而下移动上下文，接着你通过应用一个让y坐标乘以-1的缩放变换逆向y坐标的方向。

```objective-c
CGContextTranslateCTM(con, 0, theHeight);

CGContextScaleCTM(con, 1.0, -1.0);
```

上下文的顶部应该被你往下移动多远依赖于你绘制的图片。比如说我们可以绘制没有倒置问题的两个半边的火星图形（前面讨论的一个例子）。

```objective-c
CGContextTranslateCTM(con, 0, sz.height); // sz为[mars size]

CGContextScaleCTM(con, 1.0, -1.0);

CGContextDrawImage(con, CGRectMake(0, 0, sz.width/2.0, sz.height), marsLeft);

CGContextDrawImage(con, CGRectMake(b.size.width-sz.width/2.0, 0, sz.width/2.0, sz.height),marsRight);
```

## 阴影

为了在绘图上加入阴影，可在绘图之前设置上下文的阴影值。阴影的位置表示为CGSize，如果CGSize的两个值都是正数，则表示阴影是朝下和朝右的。模糊度被表示为任何一个正数。苹果没有解释缩放的工作方式，但实验表明12是最佳的模糊度，99及以上的模糊度会让阴影变得不成形。

```objective-c
con = UIGraphicsGetCurrentContext();

CGContextSetShadow(con, CGSizeMake(7, 7), 12);

[im drawAtPoint:CGPointMake(0,0)];
```
然而，使用这种方法有一个不太明显的问题。我们是在每绘制一个箭头的时候加上的阴影。因此，箭头的阴影会投射在另一个箭头上面。我们想要的是让所有的箭头集体地投射出一个阴影。解决方法是使用一个透明的图层；该图层类似一个先是叠加所有绘图然后加上阴影的一个子上下文。代码如下：

```objective-c
con = UIGraphicsGetCurrentContext();

CGContextSetShadow(con, CGSizeMake(7, 7), 12);

CGContextBeginTransparencyLayer(con, NULL);

[im drawAtPoint:CGPointMake(0,0)];

for (int i=0; i<3; i++) {

CGContextTranslateCTM(con, 20, 100);

CGContextRotateCTM(con, 30 * M_PI/180.0);

CGContextTranslateCTM(con, -20, -100);

[im drawAtPoint:CGPointMake(0,0)];

}

// 在调用了CGContextEndTransparencyLayer函数之后，

// 图层内容会在应用全局alpha和上下文阴影状态之后被合成到上下文中

CGContextEndTransparencyLayer(con);
```


## 点与像素

一个点是由xy坐标描述的一个无穷小量的位置。通过指定点实现在图形上下文中的绘图。我们并没有关心设备的分辨率，因为Core Graphics已经精细地将绘图映射到物理输出设备（基于CTM、反锯齿和平滑技术）。因此，文章之前的讨论只关心图形上下文的点，不关注点与屏幕像素的关系。

然而像素是真实存在的。一个像素是真实世界中一个具有完整物理尺寸的显示单元。整数的点实际上介于像素之间。在单分辨率设备上，这可能会让人感到迷惑。比方说，如果使用线宽为1的线条对一个整数坐标的垂直路径描边，那么线条将会被分为两半，分别落在路径的两侧。所以在单分辨率设备上线宽会变成2px（因为设备无法表示半个像素）。

当你遇到显示效果不佳的时，可能会被建议通过对坐标增减0.5让它在像素中居中。这个建议可能有效。但它只是做了一些头脑简单的假设。一个复杂的做法是获得UIView的contentScaleFactor属性。这个值为1.0或2.0，所以你可以除以这个属性值得到从像素到点的转换。还可以想想用最精确的方式绘制一条水平或垂直的线条的方式不是描边路径，而是填充路径。使用这种方法UIView的子类代码将可以在任何设备上绘制一条完美的1px宽的垂线，代码如下：

`CGContextFillRect(con, CGRectMake(100,0,1.0/self.contentScaleFactor,100));`


## 内容模式

一个视图向它自身绘图，相对于只有背景颜色和子视图，它还有内容。这意味着每当视图被调整大小它的contentMode属性就变得非常重要。正如我之前提到的，绘图系统会尽可能避免重头开始绘制视图。相反，绘图系统将使用之前绘图操作的缓存结果（位图回填）。所以，如果视图被重新调整大小，系统可能简单的伸缩或重定位缓存绘图，前提是你的contentMode设置指令是是这样设置的。

说明这一点略有点复杂。因为我需要安排调整视图大小而不引起重绘操作（调用drawRect：方法）。当程序启动时，我将创建一个MyView实例，并将它放在window上。接着将执行调整MyView尺寸的操作延迟到window出现和界面初次显示之后：

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

  self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];

  self.window.rootViewController = [UIViewController new];

  MyView* mv =[[MyView alloc] initWithFrame:CGRectMake(0, 0, self.window.bounds.size.width - 50, 150)];

  mv.center = self.window.center;

  [self.window.rootViewController.view addSubview: mv];

  mv.opaque = NO;

  mv.tag = 111; // so I can get a reference to this view later

  [self performSelector:@selector(resize:) withObject:nil afterDelay:0.1];

  self.window.backgroundColor = [UIColor whiteColor];

  [self.window makeKeyAndVisible];

  return YES;

}
```
我们将视图的高度调成之前的2倍。没有触发drawRect：方法的调用。

可是当drawRect：方法会被调用，绘图将按照drawRect：方法中的代码被刷新。代码不会将箭头绘制在相对于视图边界的高度。它是在一个固定的高度。因此箭头会伸展，而且会在以后某个时间返回到原始的尺寸。

通常我们的视图的contentMode属性需要与视图绘制自己的方式一致。假设我们的drawRect：方法中的代码让箭头的尺寸和位置相对于视图的边界原点，即它的左上方。所以我们可以设置它的contentMode为UIViewContentModeTopLeft。又或者，我们可以将contentMode设置为UIVIewContentModeRedraw，这将引起缓存内容的自动缩放和重定位被关闭，最终结果是视图的setNeedsDisplay方法将被调用，触发drawRect：方法重绘视图内容。

在另一方面，如果一个视图只是暂时被调整大小。假设是作为动画的一部分，那么伸缩行为正是你所想要的。假设我们的动画是想要让视图变大然后还原回原始大小以达到作为吸引用户的一种手段。这就需要视图伸缩的时候视图的内容也跟着伸缩，正确的contentMode的值是UIViewContentModeScaleToFill，被伸缩的内容仅仅是视图内容的一副缓存图片，所以它运行起来十分的高效。


参考文档

1. [iOS绘图教程](http://blog.csdn.net/huangznian/article/details/42741039)

