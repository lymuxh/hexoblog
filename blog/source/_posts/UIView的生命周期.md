---
title: UIView的生命周期
date: 2016-11-28 09:42:59
tags: [UIView,布局,CALayer]
---

# 创建视图

1. initWithFrame:(CGRect)frame

  initWithFrame方法用来初始化并返回一个新的视图对象,根据指定的CGRect（尺寸）。

2. initWithCoder:(NSCoder *)aDecoder

  实际编程中，**我们如果用Interface Builder 方式创建了UIView对象。（也就是，用拖控件的方式）那么，initWithFrame方法方法是不会被调用的。** 因为nib文件已经知道如何初始化该View。（因为，我们在拖该view的时候，就定义好了长、宽、背景等属性）。

  这时候，会调用initWithCoder方法，我们可以用initWithCoder方法来重新定义我们在nib中已经设置的各项属性。

<!-- more -->

3. 如何选择使用？

  当我们所写的程序里没用用Nib文件(XIB)时,用代码控制视图内容，需要调用initWithFrame去初始化

  ```objective-c
  - (id)initWithFrame:(CGRect)frame
  {
    if (self =[superinitWithFrame:frame]) {
        // 初始化代码
    }
    return self;
  }
  ```

用于视图加载nib文件，从nib中加载对象实例时，使用 initWithCoder初始化这些实例对象

  ```objective-c
  - (id)initWithCoder:(NSCoder*)coder
  {
    if (self =[superinitWithcoder:coder]) {
        // 初始化代码
    }
    return self;
  }
  ```

* initWithCoder: 对于.xib，当你嵌入一个视图对象到xib，视图加载时默认调用的是该方法；例如：假如创建的view来自nib，那么将会调用initWithCoder，由系统来调用，自己不能调用。

* initWithFrame: 非.xib的手动编码，视图加载时默认调用的是该方法。是由自己调用，来初始化对象的

# UIView的layoutSubviews和drawRect方法

 1. layoutSubviews

 在UIView里面有一个方法layoutSubviews，这个方法具体作用是什么呢？

 ```objective-c
  - (void)layoutSubviews;    // override point. called by layoutIfNeeded automatically. As of iOS 6.0, when constraints-based layout is used the base implementation applies the constraints-based layout, otherwise it does nothing.

 ```
苹果介绍如下:

 > The default implementation of this method does nothing on iOS 5.1 and earlier. Otherwise, the default implementation uses any constraints you have set to determine the size and position of any subviews.

 > Subclasses can override this method as needed to perform more precise layout of their subviews. You should override this method only if the autoresizing and constraint-based behaviors of the subviews do not offer the behavior you want. You can use your implementation to set the frame rectangles of your subviews directly.

 > You should not call this method directly. If you want to force a layout update, call the setNeedsLayout method instead to do so prior to the next drawing update. If you want to update the layout of your views immediately, call the layoutIfNeeded method.

 最后一段说，不要直接调用此方法。如果你想强制更新布局，你可以调用setNeedsLayout方法；如果你想立即数显你的views，你需要调用layoutIfNeeded方法。

 **layoutSubviews作用**

 layoutSubviews是对subviews重新布局。比如，我们想更新子视图的位置的时候，可以通过调用layoutSubviews方法，既可以实现对子视图重新布局。

 layoutSubviews默认是不做任何事情的，用到的时候，需要在自雷进行重写。

 **layoutSubviews以下情况会被调用**

 苹果官方文档已经强调，不能直接调用layoutSubviews对子视图进行重新布局。那么，layoutSubviews什么情况下会被调用呢？通过百度搜索，发现以下几种情况layoutSubviews会被调用。

 直接调用setLayoutSubviews。（这个在上面苹果官方文档里有说明）
   * addSubview的时候。
   * 当view的frame发生改变的时候。
   * 滑动UIScrollView的时候。
   * 旋转Screen会触发父UIView上的layoutSubviews事件。
   * 改变一个UIView大小的时候也会触发父UIView上的layoutSubviews事件。
   * 旋转Screen会触发父UIView上的layoutSubviews事件。

 我简单测试了一下，上面基本都会被调用。 注意：

 **当view的fram的值为0的时候，`addSubview`也不会调用`layoutSubviews`的。**

 2. drawRect:

 这个方法是用来重绘的。

 drawRect在以下情况下会被调用：

  * 如果在UIView初始化时没有设置rect大小，将直接导致drawRect不被自动调用。drawRect调用是在Controller->loadView, Controller->viewDidLoad 两方法之后掉用的.所以不用担心在控制器中,这些View的drawRect就开始画了.这样可以在控制器中设置一些值给View(如果这些View draw的时候需要用到某些变量值).

  * 该方法在调用sizeToFit后被调用，所以可以先调用sizeToFit计算出size。然后系统自动调用drawRect:方法。

  * 通过设置contentMode属性为UIViewContentModeRedraw。那么将在每次设置或更改frame的时候自动调用drawRect:。

  * 直接调用setNeedsDisplay，或者setNeedsDisplayInRect:触发drawRect:，但是有个前提条件是rect不能为0。

   ** 以上1,2推荐；而3,4不提倡 **

 drawRect方法使用注意点：

  * 若使用UIView绘图，只能在drawRect：方法中获取相应的contextRef并绘图。如果在其他方法中获取将获取到一个invalidate的ref并且不能用于画图。drawRect：方法不能手动显示调用，必须通过调用setNeedsDisplay 或者 setNeedsDisplayInRect，让系统自动调该方法。

  * 若使用calayer绘图，只能在drawInContext: 中（类似于drawRect）绘制，或者在delegate中的相应方法绘制。同样也是调用setNeedDisplay等间接调用以上方法

  * 若要实时画图，不能使用gestureRecognizer，只能使用touchbegan等方法来掉用setNeedsDisplay实时刷新屏幕

 3. sizeToFit

  * sizeToFit会自动调用sizeThatFits方法；

  * sizeToFit不应该在子类中被重写，应该重写sizeThatFits

  * sizeThatFits传入的参数是receiver当前的size，返回一个适合的size

  * sizeToFit可以被手动直接调用sizeToFit和sizeThatFits方法都没有递归，对subviews也不负责，只负责自己

# UIView和CALayer

1. 首先UIView可以响应事件，Layer不可以，父类不同。

  UIKit使用UIResponder作为响应对象，来响应系统传递过来的事件并进行处理。UIApplication、UIViewController、UIView、和所有从UIView派生出来的UIKit类（包括UIWindow）都直接或间接地继承自UIResponder类。

  在 UIResponder中定义了处理各种事件和事件传递的接口, 而 CALayer直接继承 NSObject，并没有相应的处理事件的接口。

  UIView它真正的绘图部分，是由一个叫CALayer（Core Animation Layer）的类来管理。UIView本身，更像是一个CALayer的管理器，访问它的跟绘图和跟坐标有关的属性，例如frame，bounds等等，实际上内部都是在访问它所包含的CALayer的相关属性。

  UIView有个layer属性，可以返回它的主CALayer实例，UIView有一个layerClass方法，返回主layer所使用的类，UIView的子类，可以通过重载这个方法，来让UIView使用不同的CALayer来显示。

2. 图层不会直接渲染到屏幕上。

 在模型-视图-控制器（model-view-controller）概念里面NSView和UIView是典型的视图部分，但是在核心动画里面图层是模型部分。图层封装了几何、时间、可视化属性，同时它提供了图层现实的内容，但是实际显示的过程则不是由它来完成。

 每个可见的图层树由两个相应的树组成:一个是呈现树，一个是渲染树。

3. UIView的CALayer类似UIView的子View树形结构，也可以向它的layer上添加子layer，来完成某些特殊的表示。

 ```objective-c
 grayCover = [[CALayer alloc] init];
grayCover.backgroundColor = [[[UIColor blackColor] colorWithAlphaComponent:0.2] CGColor];
[self.layer addSubLayer: grayCover];
//会在目标View上敷上一层黑色的透明薄膜。

 ```
4. UIView的layer树形在系统内部，被维护着三份copy。分别是逻辑树，这里是代码可以操纵的；动画树，是一个中间层，系统就在这一层上更改属性，进行各种渲染操作；显示树，其内容就是当前正被显示在屏幕上得内容。

5. 动画的运作：对UIView的subLayer（非主Layer）属性进行更改，系统将自动进行动画生成，动画持续时间的缺省值似乎是0.25秒。

 在 Core Animation 编程指南的 “How to Animate Layer-Backed Views” 中，对为什么会这样做出了一个解释：

 >UIView 默认情况下禁止了 layer 动画，但是在 animation block 中又重新启用了它们

 是因为任何可动画的 layer 属性改变时，layer 都会寻找并运行合适的 'action' 来实行这个改变。在 Core Animation 的专业术语中就把这样的动画统称为动作 (action，或者 CAAction)。

 layer 通过向它的 delegate 发送 actionForLayer:forKey: 消息来询问提供一个对应属性变化的 action。delegate 可以通过返回以下三者之一来进行响应：

 * 它可以返回一个动作对象，这种情况下 layer 将使用这个动作。

 * 它可以返回一个 nil， 这样 layer 就会到其他地方继续寻找。

 * 它可以返回一个 NSNull 对象，告诉 layer 这里不需要执行一个动作，搜索也会就此停止。

 当 layer 在背后支持一个 view 的时候，view 就是它的 delegate；

6. 坐标系统：CALayer的坐标系统比UIView多了一个anchorPoint属性，使用CGPoint结构表示，值域是0~1，是个比例值。这个点是各种图形变换的坐标原点，同时会更改layer的position的位置，它的缺省值是{0.5,0.5}，即在layer的中央。

 某layer.anchorPoint = CGPointMake(0.f,0.f);
如果这么设置，只会将layer的左上角被挪到原来的中间位置，必须加上这一句：
某layer.position = CGPointMake(0.f,0.f);

 最后：

 layer可以设置圆角显示（cornerRadius），也可以设置阴影 (shadowColor)。但是如果layer树中某个layer设置了圆角，树种所有layer的阴影效果都将不显示了。因此若是要有圆角又要阴影，变通方法只能做两个重叠的UIView，一个的layer显示圆角，一个layer显示阴影......

7. 渲染：当更新层，改变不能立即显示在屏幕上。当所有的层都准备好时，可以调用setNeedsDisplay方法来重绘显示。

 `[gameLayer setNeedsDisplay];`

 若要重绘部分屏幕区域，请使用setNeedsDisplayInRect:方法，通过在CGRect结构的区域更新：

 `[gameLayer setNeedsDisplayInRect:CGRectMake(150.0,100.0,50.0,75.0)];`

 如果是用的Core Graphics框架来执行渲染的话，可以直接渲染Core Graphics的内容。用renderInContext:来做这个事。

 `[gameLayer renderInContext:UIGraphicsGetCurrentContext()];`

8. 变换：要在一个层中添加一个3D或仿射变换，可以分别设置层的transform或affineTransform属性。

 ```objective-c
 characterView.layer.transform = CATransform3DMakeScale(-1.0,-1.0,1.0);

 CGAffineTransform transform = CGAffineTransformMakeRotation(45.0);
 backgroundView.layer.affineTransform = transform;
```
9. 变形：Quartz Core的渲染能力，使二维图像可以被自由操纵，就好像是三维的。图像可以在一个三维坐标系中以任意角度被旋转，缩放和倾斜。CATransform3D的一套方法提供了一些魔术般的变换效果。

# UIView autoLayout

1. iOS Autolayout

 [iOS Autolayout](http://www.jianshu.com/p/d7a4790090f1)

2. storyboard autolayout

[storyboard中autolayout和size class的使用详解](http://blog.csdn.net/liangliang103377/article/details/40082255)

3. iOS Autolayout之Masonry

[iOS Autolayout之Masonry](http://www.jianshu.com/p/10a250cc5018)


 参考目录：

1. [http://www.superqq.com/blog/2015/07/27/ioskai-fa-zhi-layoutsubviewsde-zuo-yong-he-diao-yong-ji-zhi/](http://www.superqq.com/blog/2015/07/27/ioskai-fa-zhi-layoutsubviewsde-zuo-yong-he-diao-yong-ji-zhi/)
2. [http://www.jianshu.com/p/eb2c4bb4e3f1](http://www.jianshu.com/p/eb2c4bb4e3f1)
3. [http://blog.csdn.net/weiwangchao_/article/details/7771538](http://blog.csdn.net/weiwangchao_/article/details/7771538)
4. [http://www.jianshu.com/p/079e5cf0f014](http://www.jianshu.com/p/079e5cf0f014)
