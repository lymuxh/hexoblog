---
title: CoreAnimation系列之十缓冲
date: 2016-12-07 18:18:16
tags: [CoreAnimation,缓冲]
---

在第九章“图层时间”中，我们讨论了动画时间和`CAMediaTiming`协议。现在我们来看一下另一个和时间相关的机制--所谓的*缓冲*。Core Animation使用缓冲来使动画移动更平滑更自然，而不是看起来的那种机械和人工，在这一章我们将要研究如何对你的动画控制和自定义缓冲曲线。

 <!-- more -->

## 动画速度

动画实际上就是一段时间内的变化，这就暗示了变化一定是随着某个特定的速率进行。速率由以下公式计算而来：

    velocity = change / time

这里的*变化*可以指的是一个物体移动的距离，*时间*指动画持续的时长，用这样的一个移动可以更加形象的描述（比如`position`和`bounds`属性的动画），但实际上它应用于任意可以做动画的属性（比如`color`和`opacity`）。

上面的等式假设了速度在整个动画过程中都是恒定不变的（就如同第八章“显式动画”的情况），对于这种恒定速度的动画我们称之为“线性步调”，而且从技术的角度而言这也是实现动画最简单的方式，但也是*完全不真实*的一种效果。

考虑一个场景，一辆车行驶在一定距离内，它并不会一开始就以60mph的速度行驶，然后到达终点后突然变成0mph。一是因为需要无限大的加速度（即使是最好的车也不会在0秒内从0跑到60），另外不然的话会干死所有乘客。在现实中，它会慢慢地加速到全速，然后当它接近终点的时候，它会慢慢地减速，直到最后停下来。

那么对于一个掉落到地上的物体又会怎样呢？它会首先停在空中，然后一直加速到落到地面，然后突然停止（然后由于积累的动能转换伴随着一声巨响，砰！）。

现实生活中的任何一个物体都会在运动中加速或者减速。那么我们如何在动画中实现这种加速度呢？一种方法是使用*物理引擎*来对运动物体的摩擦和动量来建模，然而这会使得计算过于复杂。我们称这种类型的方程为*缓冲函数*，幸运的是，Core Animation内嵌了一系列标准函数提供给我们使用。

### `CAMediaTimingFunction`

那么该如何使用缓冲方程式呢？首先需要设置`CAAnimation`的`timingFunction`属性，是`CAMediaTimingFunction`类的一个对象。如果想改变隐式动画的计时函数，同样也可以使用`CATransaction`的`+setAnimationTimingFunction:`方法。

这里有一些方式来创建`CAMediaTimingFunction`，最简单的方式是调用`+timingFunctionWithName:`的构造方法。这里传入如下几个常量之一：

    kCAMediaTimingFunctionLinear
    kCAMediaTimingFunctionEaseIn
    kCAMediaTimingFunctionEaseOut
    kCAMediaTimingFunctionEaseInEaseOut
    kCAMediaTimingFunctionDefault

`kCAMediaTimingFunctionLinear`选项创建了一个线性的计时函数，同样也是`CAAnimation`的`timingFunction`属性为空时候的默认函数。线性步调对于那些立即加速并且保持匀速到达终点的场景会有意义（例如射出枪膛的子弹），但是默认来说它看起来很奇怪，因为对大多数的动画来说确实很少用到。

`kCAMediaTimingFunctionEaseIn`常量创建了一个慢慢加速然后突然停止的方法。对于之前提到的自由落体的例子来说很适合，或者比如对准一个目标的导弹的发射。

`kCAMediaTimingFunctionEaseOut`则恰恰相反，它以一个全速开始，然后慢慢减速停止。它有一个削弱的效果，应用的场景比如一扇门慢慢地关上，而不是砰地一声。

`kCAMediaTimingFunctionEaseInEaseOut`创建了一个慢慢加速然后再慢慢减速的过程。这是现实世界大多数物体移动的方式，也是大多数动画来说最好的选择。如果只可以用一种缓冲函数的话，那就必须是它了。那么你会疑惑为什么这不是默认的选择，实际上当使用`UIView`的动画方法时，他的确是默认的，但当创建`CAAnimation`的时候，就需要手动设置它了。

最后还有一个`kCAMediaTimingFunctionDefault`，它和`kCAMediaTimingFunctionEaseInEaseOut`很类似，但是加速和减速的过程都稍微有些慢。它和`kCAMediaTimingFunctionEaseInEaseOut`的区别很难察觉，可能是苹果觉得它对于隐式动画来说更适合（然后对UIKit就改变了想法，而是使用`kCAMediaTimingFunctionEaseInEaseOut`作为默认效果），虽然它的名字说是默认的，但还是要记住当创建*显式*的`CAAnimation`它并不是默认选项（换句话说，默认的图层行为动画用`kCAMediaTimingFunctionDefault`作为它们的计时方法）。

你可以使用一个简单的测试工程来实验一下，在运行之前改变缓冲函数的代码，然后点击任何地方来观察图层是如何通过指定的缓冲移动的。

```Objective-C
@interface ViewController ()

@property (nonatomic, strong) CALayer *colorLayer;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //create a red layer
    self.colorLayer = [CALayer layer];
    self.colorLayer.frame = CGRectMake(0, 0, 100, 100);
    self.colorLayer.position = CGPointMake(self.view.bounds.size.width/2.0, self.view.bounds.size.height/2.0);
    self.colorLayer.backgroundColor = [UIColor redColor].CGColor;
    [self.view.layer addSublayer:self.colorLayer];
}

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    //configure the transaction
    [CATransaction begin];
    [CATransaction setAnimationDuration:1.0];
    [CATransaction setAnimationTimingFunction:[CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseOut]];
    //set the position
    self.colorLayer.position = [[touches anyObject] locationInView:self.view];
    //commit transaction
    [CATransaction commit];
}

@end
```

### `UIView`的动画缓冲

UIKit的动画也同样支持这些缓冲方法的使用，尽管语法和常量有些不同，为了改变`UIView`动画的缓冲选项，给`options`参数添加如下常量之一：

    UIViewAnimationOptionCurveEaseInOut
    UIViewAnimationOptionCurveEaseIn
    UIViewAnimationOptionCurveEaseOut
    UIViewAnimationOptionCurveLinear

它们和`CAMediaTimingFunction`紧密关联，`UIViewAnimationOptionCurveEaseInOut`是默认值（这里没有`kCAMediaTimingFunctionDefault`相对应的值了）。


```objective-c
@interface ViewController ()

@property (nonatomic, strong) UIView *colorView;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //create a red layer
    self.colorView = [[UIView alloc] init];
    self.colorView.bounds = CGRectMake(0, 0, 100, 100);
    self.colorView.center = CGPointMake(self.view.bounds.size.width / 2, self.view.bounds.size.height / 2);
    self.colorView.backgroundColor = [UIColor redColor];
    [self.view addSubview:self.colorView];
}

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    //perform the animation
    [UIView animateWithDuration:1.0 delay:0.0
                        options:UIViewAnimationOptionCurveEaseOut
                     animations:^{
                            //set the position
                            self.colorView.center = [[touches anyObject] locationInView:self.view];
                        }
                     completion:NULL];

}

@end
```

### 缓冲和关键帧动画

或许你会回想起第八章里面颜色切换的关键帧动画由于线性变换的原因（见清单8.5）看起来有些奇怪，使得颜色变换非常不自然。为了纠正这点，我们来用更加合适的缓冲方法，例如`kCAMediaTimingFunctionEaseIn`，给图层的颜色变化添加一点*脉冲*效果，让它更像现实中的一个彩色灯泡。

我们不想给整个动画过程应用这个效果，我们希望对每个动画的过程重复这样的缓冲，于是每次颜色的变换都会有脉冲效果。

`CAKeyframeAnimation`有一个`NSArray`类型的`timingFunctions`属性，我们可以用它来对每次动画的步骤指定不同的计时函数。但是指定函数的个数一定要等于`keyframes`数组的元素个数*减一*，因为它是描述每一帧之间动画速度的函数。

在这个例子中，我们自始至终想使用同一个缓冲函数，但我们同样需要一个函数的数组来告诉动画不停地重复每个步骤，而不是在整个动画序列只做一次缓冲，我们简单地使用包含多个相同函数拷贝的数组就可以了。

运行更新后的代码，你会发现动画看起来更加自然了。


```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *layerView;
@property (nonatomic, weak) IBOutlet CALayer *colorLayer;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //create sublayer
    self.colorLayer = [CALayer layer];
    self.colorLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
    self.colorLayer.backgroundColor = [UIColor blueColor].CGColor;
    //add it to our view
    [self.layerView.layer addSublayer:self.colorLayer];
}

- (IBAction)changeColor
{
    //create a keyframe animation
    CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
    animation.keyPath = @"backgroundColor";
    animation.duration = 2.0;
    animation.values = @[
                         (__bridge id)[UIColor blueColor].CGColor,
                         (__bridge id)[UIColor redColor].CGColor,
                         (__bridge id)[UIColor greenColor].CGColor,
                         (__bridge id)[UIColor blueColor].CGColor ];
    //add timing function
    CAMediaTimingFunction *fn = [CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseIn];
    animation.timingFunctions = @[fn, fn, fn];
    //apply animation to layer
    [self.colorLayer addAnimation:animation forKey:nil];
}

@end
```
## 自定义缓冲函数

在第八章中，我们给时钟项目添加了动画。看起来很赞，但是如果有合适的缓冲函数就更好了。在显示世界中，钟表指针转动的时候，通常起步很慢，然后迅速啪地一声，最后缓冲到终点。但是标准的缓冲函数在这里每一个适合它，那该如何创建一个新的呢？

除了`+functionWithName:`之外，`CAMediaTimingFunction`同样有另一个构造函数，一个有四个浮点参数的`+functionWithControlPoints::::`（注意这里奇怪的语法，并没有包含具体每个参数的名称，这在objective-C中是合法的，但是却违反了苹果对方法命名的指导方针，而且看起来是一个奇怪的设计）。

使用这个方法，我们可以创建一个自定义的缓冲函数，来匹配我们的时钟动画，为了理解如何使用这个方法，我们要了解一些`CAMediaTimingFunction`是如何工作的。

### 三次贝塞尔曲线

`CAMediaTimingFunction`函数的主要原则在于它把输入的时间转换成起点和终点之间成比例的改变。我们可以用一个简单的图标来解释，横轴代表时间，纵轴代表改变的量，于是线性的缓冲就是一条从起点开始的简单的斜线。

这条曲线的斜率代表了速度，斜率的改变代表了加速度，原则上来说，任何加速的曲线都可以用这种图像来表示，但是`CAMediaTimingFunction`使用了一个叫做*三次贝塞尔曲线*的函数，它只可以产出指定缓冲函数的子集（我们之前在第八章中创建`CAKeyframeAnimation`路径的时候提到过三次贝塞尔曲线）。

你或许会回想起，一个三次贝塞尔曲线通过四个点来定义，第一个和最后一个点代表了曲线的起点和终点，剩下中间两个点叫做*控制点*，因为它们控制了曲线的形状，贝塞尔曲线的控制点其实是位于曲线之外的点，也就是说曲线并不一定要穿过它们。你可以把它们想象成吸引经过它们曲线的磁铁。

实际上它是一个很奇怪的函数，先加速，然后减速，最后快到达终点的时候又加速，那么标准的缓冲函数又该如何用图像来表示呢？

`CAMediaTimingFunction`有一个叫做`-getControlPointAtIndex:values:`的方法，可以用来检索曲线的点，这个方法的设计的确有点奇怪（或许也就只有苹果能回答为什么不简单返回一个`CGPoint`），但是使用它我们可以找到标准缓冲函数的点，然后用`UIBezierPath`和`CAShapeLayer`来把它画出来。

曲线的起始和终点始终是{0, 0}和{1, 1}，于是我们只需要检索曲线的第二个和第三个点（控制点）。

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *layerView;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //create timing function
    CAMediaTimingFunction *function = CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseOut];
    //get control points
    CGPoint controlPoint1, controlPoint2;
    [function getControlPointAtIndex:1 values:(float *)&controlPoint1];
    [function getControlPointAtIndex:2 values:(float *)&controlPoint2];
    //create curve
    UIBezierPath *path = [[UIBezierPath alloc] init];
    [path moveToPoint:CGPointZero];
    [path addCurveToPoint:CGPointMake(1, 1)
            controlPoint1:controlPoint1 controlPoint2:controlPoint2];
    //scale the path up to a reasonable size for display
    [path applyTransform:CGAffineTransformMakeScale(200, 200)];
    //create shape layer
    CAShapeLayer *shapeLayer = [CAShapeLayer layer];
    shapeLayer.strokeColor = [UIColor redColor].CGColor;
    shapeLayer.fillColor = [UIColor clearColor].CGColor;
    shapeLayer.lineWidth = 4.0f;
    shapeLayer.path = path.CGPath;
    [self.layerView.layer addSublayer:shapeLayer];
    //flip geometry so that 0,0 is in the bottom-left
    self.layerView.layer.geometryFlipped = YES;
}

@end
```

那么对于我们自定义时钟指针的缓冲函数来说，我们需要初始微弱，然后迅速上升，最后缓冲到终点的曲线，通过一些实验之后，最终结果如下：

    [CAMediaTimingFunction functionWithControlPoints:1 :0 :0.75 :1];

    ```objective-c
    - (void)setAngle:(CGFloat)angle forHand:(UIView *)handView animated:(BOOL)animated
    {
        //generate transform
        CATransform3D transform = CATransform3DMakeRotation(angle, 0, 0, 1);
        if (animated) {
            //create transform animation
            CABasicAnimation *animation = [CABasicAnimation animation];
            animation.keyPath = @"transform";
            animation.fromValue = [handView.layer.presentationLayer valueForKey:@"transform"];
            animation.toValue = [NSValue valueWithCATransform3D:transform];
            animation.duration = 0.5;
            animation.delegate = self;
            animation.timingFunction = [CAMediaTimingFunction functionWithControlPoints:1 :0 :0.75 :1];
            //apply animation
            handView.layer.transform = transform;
            [handView.layer addAnimation:animation forKey:nil];
        } else {
            //set transform directly
            handView.layer.transform = transform;
        }
    }
    ```

### 更加复杂的动画曲线

考虑一个橡胶球掉落到坚硬的地面的场景，当开始下落的时候，它会持续加速知道落到地面，然后经过几次反弹，最后停下来。如果用一张图来说明。
这种效果没法用一个简单的三次贝塞尔曲线表示，于是不能用`CAMediaTimingFunction`来完成。但如果想要实现这样的效果，可以用如下几种方法：

* 用`CAKeyframeAnimation`创建一个动画，然后分割成几个步骤，每个小步骤使用自己的计时函数（具体下节介绍）。
* 使用定时器逐帧更新实现动画（见第11章，“基于定时器的动画”）。

### 基于关键帧的缓冲

为了使用关键帧实现反弹动画，我们需要在缓冲曲线中对每一个显著的点创建一个关键帧（在这个情况下，关键点也就是每次反弹的峰值），然后应用缓冲函数把每段曲线连接起来。同时，我们也需要通过`keyTimes`来指定每个关键帧的时间偏移，由于每次反弹的时间都会减少，于是关键帧并不会均匀分布。


```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *containerView;
@property (nonatomic, strong) UIImageView *ballView;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //add ball image view
    UIImage *ballImage = [UIImage imageNamed:@"Ball.png"];
    self.ballView = [[UIImageView alloc] initWithImage:ballImage];
    [self.containerView addSubview:self.ballView];
    //animate
    [self animate];
}

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    //replay animation on tap
    [self animate];
}

- (void)animate
{
    //reset ball to top of screen
    self.ballView.center = CGPointMake(150, 32);
    //create keyframe animation
    CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
    animation.keyPath = @"position";
    animation.duration = 1.0;
    animation.delegate = self;
    animation.values = @[
                         [NSValue valueWithCGPoint:CGPointMake(150, 32)],
                         [NSValue valueWithCGPoint:CGPointMake(150, 268)],
                         [NSValue valueWithCGPoint:CGPointMake(150, 140)],
                         [NSValue valueWithCGPoint:CGPointMake(150, 268)],
                         [NSValue valueWithCGPoint:CGPointMake(150, 220)],
                         [NSValue valueWithCGPoint:CGPointMake(150, 268)],
                         [NSValue valueWithCGPoint:CGPointMake(150, 250)],
                         [NSValue valueWithCGPoint:CGPointMake(150, 268)]
                         ];

    animation.timingFunctions = @[
                                  [CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseIn],
                                  [CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseOut],
                                  [CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseIn],
                                  [CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseOut],
                                  [CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseIn],
                                  [CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseOut],
                                  [CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseIn]
                                  ];

    animation.keyTimes = @[@0.0, @0.3, @0.5, @0.7, @0.8, @0.9, @0.95, @1.0];
    //apply animation
    self.ballView.layer.position = CGPointMake(150, 268);
    [self.ballView.layer addAnimation:animation forKey:nil];
}

@end
```
这种方式还算不错，但是实现起来略显笨重（因为要不停地尝试计算各种关键帧和时间偏移）并且和动画强绑定了（因为如果要改变动画的一个属性，那就意味着要重新计算所有的关键帧）。那该如何写一个方法，用缓冲函数来把任何简单的属性动画转换成关键帧动画呢，下面我们来实现它。

### 流程自动化

上面代码我们把动画分割成相当大的几块，然后用Core Animation的缓冲进入和缓冲退出函数来大约形成我们想要的曲线。但如果我们把动画分割成更小的几部分，那么我们就可以用直线来拼接这些曲线（也就是线性缓冲）。为了实现自动化，我们需要知道如何做如下两件事情：

* 自动把任意属性动画分割成多个关键帧
* 用一个数学函数表示弹性动画，使得可以对帧做便宜

为了解决第一个问题，我们需要复制Core Animation的插值机制。这是一个传入起点和终点，然后在这两个点之间指定时间点产出一个新点的机制。对于简单的浮点起始值，公式如下（假设时间从0到1）：

    value = (endValue – startValue) × time + startValue;

那么如果要插入一个类似于`CGPoint`，`CGColorRef`或者`CATransform3D`这种更加复杂类型的值，我们可以简单地对每个独立的元素应用这个方法（也就`CGPoint`中的x和y值，`CGColorRef`中的红，蓝，绿，透明值，或者是`CATransform3D`中独立矩阵的坐标）。我们同样需要一些逻辑在插值之前对对象拆解值，然后在插值之后在重新封装成对象，也就是说需要实时地检查类型。

一旦我们可以用代码获取属性动画的起始值之间的任意插值，我们就可以把动画分割成许多独立的关键帧，然后产出一个线性的关键帧动画。下面展示了相关代码。

注意到我们用了60 x 动画时间（秒做单位）作为关键帧的个数，这时因为Core Animation按照每秒60帧去渲染屏幕更新，所以如果我们每秒生成60个关键帧，就可以保证动画足够的平滑（尽管实际上很可能用更少的帧率就可以达到很好的效果）。

我们在示例中仅仅引入了对`CGPoint`类型的插值代码。但是，从代码中很清楚能看出如何扩展成支持别的类型。作为不能识别类型的备选方案，我们仅仅在前一半返回了`fromValue`，在后一半返回了`toValue`。

```objective-c
float interpolate(float from, float to, float time)
{
    return (to - from) * time + from;
}

- (id)interpolateFromValue:(id)fromValue toValue:(id)toValue time:(float)time
{
    if ([fromValue isKindOfClass:[NSValue class]]) {
        //get type
        const char *type = [fromValue objCType];
        if (strcmp(type, @encode(CGPoint)) == 0) {
            CGPoint from = [fromValue CGPointValue];
            CGPoint to = [toValue CGPointValue];
            CGPoint result = CGPointMake(interpolate(from.x, to.x, time), interpolate(from.y, to.y, time));
            return [NSValue valueWithCGPoint:result];
        }
    }
    //provide safe default implementation
    return (time < 0.5)? fromValue: toValue;
}

- (void)animate
{
    //reset ball to top of screen
    self.ballView.center = CGPointMake(150, 32);
    //set up animation parameters
    NSValue *fromValue = [NSValue valueWithCGPoint:CGPointMake(150, 32)];
    NSValue *toValue = [NSValue valueWithCGPoint:CGPointMake(150, 268)];
    CFTimeInterval duration = 1.0;
    //generate keyframes
    NSInteger numFrames = duration * 60;
    NSMutableArray *frames = [NSMutableArray array];
    for (int i = 0; i < numFrames; i++) {
        float time = 1 / (float)numFrames * i;
        [frames addObject:[self interpolateFromValue:fromValue toValue:toValue time:time]];
    }
    //create keyframe animation
    CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
    animation.keyPath = @"position";
    animation.duration = 1.0;
    animation.delegate = self;
    animation.values = frames;
    //apply animation
    [self.ballView.layer addAnimation:animation forKey:nil];
}
```
这可以起到作用，但效果并不是很好，到目前为止我们所完成的只是一个非常复杂的方式来使用线性缓冲复制`CABasicAnimation`的行为。这种方式的好处在于我们可以更加精确地控制缓冲，这也意味着我们可以应用一个完全定制的缓冲函数。那么该如何做呢？

缓冲背后的数学并不很简单，但是幸运的是我们不需要一一实现它。罗伯特·彭纳有一个网页关于缓冲函数（[http://www.robertpenner.com/easing](http://www.robertpenner.com/easing)），包含了大多数普遍的缓冲函数的多种编程语言的实现的链接，包括C。这里是一个缓冲进入缓冲退出函数的示例（实际上有很多不同的方式去实现它）。

```c
float quadraticEaseInOut(float t)
{
    return (t < 0.5)? (2 * t * t): (-2 * t * t) + (4 * t) - 1;
}
```

对我们的弹性球来说，我们可以使用`bounceEaseOut`函数：

```c
float bounceEaseOut(float t)
{
    if (t < 4/11.0) {
        return (121 * t * t)/16.0;
    } else if (t < 8/11.0) {
        return (363/40.0 * t * t) - (99/10.0 * t) + 17/5.0;
    } else if (t < 9/10.0) {
        return (4356/361.0 * t * t) - (35442/1805.0 * t) + 16061/1805.0;
    }
    return (54/5.0 * t * t) - (513/25.0 * t) + 268/25.0;
}
```

如果修改上面的代码来引入`bounceEaseOut`方法，我们的任务就是仅仅交换缓冲函数，现在就可以选择任意的缓冲类型创建动画了。

以下用关键帧实现自定义的缓冲函数。

```objective-c
- (void)animate
{
    //reset ball to top of screen
    self.ballView.center = CGPointMake(150, 32);
    //set up animation parameters
    NSValue *fromValue = [NSValue valueWithCGPoint:CGPointMake(150, 32)];
    NSValue *toValue = [NSValue valueWithCGPoint:CGPointMake(150, 268)];
    CFTimeInterval duration = 1.0;
    //generate keyframes
    NSInteger numFrames = duration * 60;
    NSMutableArray *frames = [NSMutableArray array];
    for (int i = 0; i < numFrames; i++) {
        float time = 1/(float)numFrames * i;
        //apply easing
        time = bounceEaseOut(time);
        //add keyframe
        [frames addObject:[self interpolateFromValue:fromValue toValue:toValue time:time]];
    }
    //create keyframe animation
    CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
    animation.keyPath = @"position";
    animation.duration = 1.0;
    animation.delegate = self;
    animation.values = frames;
    //apply animation
    [self.ballView.layer addAnimation:animation forKey:nil];
}
```

参考文章：
1. [http://blog.csdn.net/huangznian/article/details/42918393](http://blog.csdn.net/huangznian/article/details/42918393)
