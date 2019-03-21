---
title: UITableView的性能优化
date: 2016-11-29 11:11:10
tags: [UITableView,性能优化,随笔,文档]
---

# UITableView介绍

在iOS开发中UITableView可以说是使用最广泛的控件，我们平时使用的软件中到处都可以看到它的影子，类似于微信、QQ、新浪微博等软件基本上随处都是UITableView。当然它的广泛使用自然离不开它强大的功能，今天这篇文章将针对UITableView重点展开讨论。

UITableView有两种风格：UITableViewStylePlain和UITableViewStyleGrouped。这两者操作起来其实并没有本质区别，只是后者按分组样式显示前者按照普通样式显示而已。

UITableView中每行数据都是一个UITableViewCell，在这个控件中为了显示更多的信息，iOS已经在其内部设置好了多个子控件以供开发者使用。如果我们查看UITableViewCell的声明文件可以发现在内部有一个UIView控件（contentView，作为其他元素的父控件）、两个UILable控件（textLabel、detailTextLabel）、一个UIImage控件（imageView），分别用于容器、显示内容、详情和图片。

<!-- more -->

#  UITableView 数据源

 由于iOS是遵循MVC模式设计的，很多操作都是通过代理和外界沟通的，但对于数据源控件除了代理还有一个数据源属性，通过它和外界进行数据交互。 对于UITableView设置完dataSource后需要实现UITableViewDataSource协议，在这个协议中定义了多种 数据操作方法。

```objective-c

 @required
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section;

// Row display. Implementers should *always* try to reuse cells by setting each cell's reuseIdentifier and querying for available reusable cells with dequeueReusableCellWithIdentifier:

行显示，重写应该始终尝试通过设置行的重用标示进行复用，通过dequeueReusableCellWithIdentifier：来进行可用行的查询。

// Cell gets various attributes set automatically based on table (separators) and data source (accessory views, editing controls)

根据表的数据自动设置对应行的各种显示变量和辅助视图，编辑控件.

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath;

@optional

- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView;              // Default is 1 if not implemented
分组数设置，默认为1.
- (nullable NSString *)tableView:(UITableView *)tableView titleForHeaderInSection:(NSInteger)section;
分组头部信息设置
// fixed font style. use custom view (UILabel) if you want something different

- (nullable NSString *)tableView:(UITableView *)tableView titleForFooterInSection:(NSInteger)section;
分组底部信息设置
// Editing

// Individual rows can opt out of having the -editing property set for them. If not implemented, all rows are assumed to be editable.
每行都可以独立设置编辑属性，默认假定都是可编辑的。
- (BOOL)tableView:(UITableView *)tableView canEditRowAtIndexPath:(NSIndexPath *)indexPath;

// Moving/reordering
移动／排序
// Allows the reorder accessory view to optionally be shown for a particular row. By default, the reorder control will be shown only if the datasource implements -tableView:moveRowAtIndexPath:toIndexPath:
允许显示的行进行重新排序，通过委托-tableView:moveRowAtIndexPath:toIndexPath:进行编辑控件显示。

- (BOOL)tableView:(UITableView *)tableView canMoveRowAtIndexPath:(NSIndexPath *)indexPath;
设置行是否可以移动
// Index

- (nullable NSArray<NSString *> *)sectionIndexTitlesForTableView:(UITableView *)tableView
 __TVOS_PROHIBITED;                                                    // return list of section titles to display in section index view (e.g. "ABCD...Z#")
 设置分组索引

- (NSInteger)tableView:(UITableView *)tableView sectionForSectionIndexTitle:(NSString *)title atIndex:(NSInteger)index __TVOS_PROHIBITED;
 // tell table which section corresponds to section title/index (e.g. "B",1))

告诉列表哪个分组对应的标题／索引相符。

// Data manipulation - insert and delete support
数据添加和删除支持

// After a row has the minus or plus button invoked (based on the UITableViewCellEditingStyle for the cell), the dataSource must commit the change
// Not called for edit actions using UITableViewRowAction - the action's handler will be invoked instead

行根据UITableViewCellEditingStyle设置添加和删除按钮，数据源必须提交变化。

- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath;

指定行编辑的提交。
// Data manipulation - reorder / moving support

- (void)tableView:(UITableView *)tableView moveRowAtIndexPath:(NSIndexPath *)sourceIndexPath toIndexPath:(NSIndexPath *)destinationIndexPath;
指定位置行移动到另一个位置

  ```

# UITableView 代理

```objective-c
@protocol UITableViewDelegate<NSObject, UIScrollViewDelegate>

@optional

// Display customization
自定义显示的回调，包含准备显示和显示完毕
- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath;

- (void)tableView:(UITableView *)tableView willDisplayHeaderView:(UIView *)view forSection:(NSInteger)section NS_AVAILABLE_IOS(6_0);

- (void)tableView:(UITableView *)tableView willDisplayFooterView:(UIView *)view forSection:(NSInteger)section NS_AVAILABLE_IOS(6_0);

- (void)tableView:(UITableView *)tableView didEndDisplayingCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath*)indexPath NS_AVAILABLE_IOS(6_0);

- (void)tableView:(UITableView *)tableView didEndDisplayingHeaderView:(UIView *)view forSection:(NSInteger)section NS_AVAILABLE_IOS(6_0);

- (void)tableView:(UITableView *)tableView didEndDisplayingFooterView:(UIView *)view forSection:(NSInteger)section NS_AVAILABLE_IOS(6_0);

// Variable height support
 高度设置，包含头部、底部还有行高度

- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath;
- (CGFloat)tableView:(UITableView *)tableView heightForHeaderInSection:(NSInteger)section;
- (CGFloat)tableView:(UITableView *)tableView heightForFooterInSection:(NSInteger)section;

// Use the estimatedHeight methods to quickly calcuate guessed values which will allow for fast load times of the table.
使用估算高度方法可以快速计算高度值进而方便表格快速加载。
// If these methods are implemented, the above -tableView:heightForXXX calls will be deferred until views are ready to be displayed, so more expensive logic can be placed there.
如果重写这些方法上面的方法会在将要显示的时候调用，这样更多复杂逻辑可以放到里面。

- (CGFloat)tableView:(UITableView *)tableView estimatedHeightForRowAtIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(7_0);
-
- (CGFloat)tableView:(UITableView *)tableView estimatedHeightForHeaderInSection:(NSInteger)section NS_AVAILABLE_IOS(7_0);
-
- (CGFloat)tableView:(UITableView *)tableView estimatedHeightForFooterInSection:(NSInteger)section NS_AVAILABLE_IOS(7_0);

// Section header & footer information. Views are preferred over title should you decide to provide both
分组的头部和底部信息，视图默认会覆盖标题方法。
- (nullable UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section;
// custom view for header. will be adjusted to default or specified header height
返回自定义头部视图。会适配默认或者指定高度。

- (nullable UIView *)tableView:(UITableView *)tableView viewForFooterInSection:(NSInteger)section;
// custom view for footer. will be adjusted to default or specified footer height
返回自定义底部部视图。会适配默认或者指定高度。
// Accessories (disclosures).

- (UITableViewCellAccessoryType)tableView:(UITableView *)tableView accessoryTypeForRowWithIndexPath:(NSIndexPath *)indexPath NS_DEPRECATED_IOS(2_0, 3_0) __TVOS_PROHIBITED;
返回自定义扩展视图类型。

- (void)tableView:(UITableView *)tableView accessoryButtonTappedForRowWithIndexPath:(NSIndexPath *)indexPath;
行扩展视图点击事件处理
// Selection

// -tableView:shouldHighlightRowAtIndexPath: is called when a touch comes down on a row.
点击一行是否显示高亮
// Returning NO to that message halts the selection process and does not cause the currently selected row to lose its selected look while the touch is down.
返回NO，当点击时阻止选择处理过程，但是不会导致当前行选择状态变化。

- (BOOL)tableView:(UITableView *)tableView shouldHighlightRowAtIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(6_0);

- (void)tableView:(UITableView *)tableView didHighlightRowAtIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(6_0);

- (void)tableView:(UITableView *)tableView didUnhighlightRowAtIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(6_0);

// Called before the user changes the selection. Return a new indexPath, or nil, to change the proposed selection.
行选择变化前调用。返回一个indexPath或者nil。

- (nullable NSIndexPath *)tableView:(UITableView *)tableView willSelectRowAtIndexPath:(NSIndexPath *)indexPath;

- (nullable NSIndexPath *)tableView:(UITableView *)tableView willDeselectRowAtIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(3_0);

// Called after the user changes the selection.
行选择变化后调用。返回一个indexPath或者nil。

- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath;

- (void)tableView:(UITableView *)tableView didDeselectRowAtIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(3_0);

// Editing
编辑
// Allows customization of the editingStyle for a particular cell located at 'indexPath'. If not implemented, all editable cells will have UITableViewCellEditingStyleDelete set for them when the table has editing property set to YES.
允许指定行自定义编辑分割，如果没有实现，当编辑属性为YES的话，默认行支持删除。
- (UITableViewCellEditingStyle)tableView:(UITableView *)tableView editingStyleForRowAtIndexPath:(NSIndexPath *)indexPath;

- (nullable NSString *)tableView:(UITableView *)tableView titleForDeleteConfirmationButtonForRowAtIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(3_0) __TVOS_PROHIBITED;

- (nullable NSArray<UITableViewRowAction *> *)tableView:(UITableView *)tableView editActionsForRowAtIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(8_0) __TVOS_PROHIBITED;
// supercedes -tableView:titleForDeleteConfirmationButtonForRowAtIndexPath: if return value is non-nil
指定行删除确定标题
// Controls whether the background is indented while editing.  If not implemented, the default is YES.  This is unrelated to the indentation level below.  This method only applies to grouped style table views.
控制编辑时是否缩近，默认是YES。这个和缩进的程度没有关系。
- (BOOL)tableView:(UITableView *)tableView shouldIndentWhileEditingRowAtIndexPath:(NSIndexPath *)indexPath;

// The willBegin/didEnd methods are called whenever the 'editing' property is automatically changed by the table (allowing insert/delete/move). This is done by a swipe activating a single row
下面两个方法当表格编辑状态变化时会自动调用。通过滑动一行来激活编辑状态。
- (void)tableView:(UITableView *)tableView willBeginEditingRowAtIndexPath:(NSIndexPath *)indexPath __TVOS_PROHIBITED;

- (void)tableView:(UITableView *)tableView didEndEditingRowAtIndexPath:(nullable NSIndexPath *)indexPath __TVOS_PROHIBITED;

// Moving/reordering

// Allows customization of the target row for a particular row as it is being moved/reordered
允许某一行移动到其他行。
- (NSIndexPath *)tableView:(UITableView *)tableView targetIndexPathForMoveFromRowAtIndexPath:(NSIndexPath *)sourceIndexPath toProposedIndexPath:(NSIndexPath *)proposedDestinationIndexPath;

// Indentation

- (NSInteger)tableView:(UITableView *)tableView indentationLevelForRowAtIndexPath:(NSIndexPath *)indexPath; // return 'depth' of row for hierarchies

// Copy/Paste.  All three methods must be implemented by the delegate.
所有方法必须实现
- (BOOL)tableView:(UITableView *)tableView shouldShowMenuForRowAtIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(5_0);
- (BOOL)tableView:(UITableView *)tableView canPerformAction:(SEL)action forRowAtIndexPath:(NSIndexPath *)indexPath withSender:(nullable id)sender NS_AVAILABLE_IOS(5_0);
- (void)tableView:(UITableView *)tableView performAction:(SEL)action forRowAtIndexPath:(NSIndexPath *)indexPath withSender:(nullable id)sender NS_AVAILABLE_IOS(5_0);

// Focus

- (BOOL)tableView:(UITableView *)tableView canFocusRowAtIndexPath:(NSIndexPath *)indexPath NS_AVAILABLE_IOS(9_0);
- (BOOL)tableView:(UITableView *)tableView shouldUpdateFocusInContext:(UITableViewFocusUpdateContext *)context NS_AVAILABLE_IOS(9_0);
- (void)tableView:(UITableView *)tableView didUpdateFocusInContext:(UITableViewFocusUpdateContext *)context withAnimationCoordinator:(UIFocusAnimationCoordinator *)coordinator NS_AVAILABLE_IOS(9_0);
- (nullable NSIndexPath *)indexPathForPreferredFocusedViewInTableView:(UITableView *)tableView NS_AVAILABLE_IOS(9_0);

@end
```

# UITableView大量数据滚动性能优化

 UITableView中的单元格cell是在显示到用户可视区域后创建的，那么如果用户往下滚动就会继续创建显示在屏幕上的单元格，如果用户向上滚动返回到查看过的内容时同样会重新创建之前已经创建过的单元格。如此一来即使UITableView的内容不是太多，如果用户反复的上下滚动，内存也会瞬间飙升，更何况很多时候UITableView的内容是很多的（例如微博展示列表，基本向下滚动是没有底限的）。

`- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath`

这个代理方法的实现，在可见的页面是会重复绘制页面的，所以绝大部分人都会在这里做一些代码处理
比如：
```objective-c
static NSString *CellIdentifier = @"LazyTableCell";
UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:CellIdentifier];
```
以上举例代码是可以让cell被重复使用，一般大概只会在可见页面部分的几个cell会被new下，其他的全部重复使用前面已经有的cell对象，到时候只要填充数据就可以了。

那么仅仅只是如此，恐怕现在的cell自定义的页面不只是文本那么简单，多多少少都会带有一些图片吧，当你下滑时候是否发现有那么一点点的卡顿现成，特别是网络不好，而且还是在iPhone5上跑的就会更明显了。

那么在cell里面异步加载图片是个程序员都会想到，但是如果你给每个循环对象都加上异步加载，并且下滑的时候，这一操作将会被执行，虽然是异步，但是一个app里面的线程过多也会卡顿的，特别是在下滑操作的时候给每个图片进行异步加载。

那么这里可以利用UIScrollViewDelegate代理很好的解决这问题

```objective-c
- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate

- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView

```
可以识别tableview静止或者减速滑动结束的时候进行异步加载图片

以下方法来执行异步加载操作

```objective-c
      //获取可见部分的对象
       NSArray *visiblePaths = [self.tableView indexPathsForVisibleRows];
        for (NSIndexPath *indexPath in visiblePaths)
        {
           //获取的dataSource里面的对象，并且判断加载完成的不需要再次异步加载
             <code>
        }
```

同时在cell绘制中也做限制

```objective-c
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath

         if (self.tableView.dragging == NO && self.tableView.decelerating == NO)
            {
               //开始异步加载图片
                <code>
            }
```

如果tableview 停止滑动的时候开始异步加载图片。

最后也别忘记在内存紧张的情况下释放调所有的异步线程，以保证的你的app不会被系统强制关闭。

```objective-c
- (void)didReceiveMemoryWarning{
//  释放调异步加载图片的线程以及所有图片资源对象
<code>
}
```

还有千万别忘记销毁的时候手动把所有的使用到的代理设置nil.

# UITableView中NSTime停止的问题

在Windows时代，大家肯定对SendMessage，PostMessage，GetMessage有所了解，这些都是windows中的消息处理函数，那对应在ios中是什么呢，其实就是NSRunloop这个东西。在ios中，所有消息都会被添加到NSRunloop中，分为‘input source’跟'timer source'种，并在循环中检查是不是有事件需要发生，如果需要那么就调用相应的函数处理。

我们在使用NSTimer的时候，可能会接触到runloop的概念，下面是一个简单的例子：

```objective-c
    NSTimer * timer = [NSTimer scheduledTimerWithTimeInterval:1
                                              target:self
                                             selector:@selector(printMessage:)
                                            userInfo:nil
                                             repeats:YES];
```

这个时候如果我们在界面上滚动一个scrollview，那么我们会发现在停止滚动前，控制台不会有任何输出，就好像scrollView在滚动的时候将timer暂停了一样，在查看相应文档后发现，这其实就是runloop的mode在做怪。

runloop可以理解为cocoa下的一种消息循环机制，用来处理各种消息事件，我们在开发的时候并不需要手动去创建一个runloop，因为框架为我们创建了一个默认的runloop,通过[NSRunloop currentRunloop]我们可以得到一个当前线程下面对应的runloop对象，不过我们需要注意的是不同的runloop之间消息的通知方式。

接着上面的话题，在开启一个NSTimer实质上是在当前的runloop中注册了一个新的事件源，而当scrollView滚动的时候，当前的MainRunLoop是处于UITrackingRunLoopMode的模式下，在这个模式下，是不会处理NSDefaultRunLoopMode的消息(因为RunLoop Mode不一样)，要想在scrollView滚动的同时也接受其它runloop的消息，我们需要改变两者之间的runloopmode.

`[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];`


简单的说就是NSTimer不会开启新的进程，只是在Runloop里注册了一下，Runloop每次loop时都会检测这个timer，看是否可以触发。当Runloop在A mode，而timer注册在B mode时就无法去检测这个timer，所以需要把NSTimer也注册到A mode，这样就可以被检测到。

说到这里，在http异步通信的模块中也有可能碰到这样的问题，就是在向服务器异步获取图片数据通知主线程刷新tableView中的图片时，在tableView滚动没有停止或用户手指停留在屏幕上的时候，图片一直不会出来，可能背后也是这个runloop的mode在做怪，嘿嘿。


参考文章

1. [http://www.cocoachina.com/ios/20140922/9710.html](http://www.cocoachina.com/ios/20140922/9710.html)
2. [http://bbs.51cto.com/thread-1123666-1-1.html](http://bbs.51cto.com/thread-1123666-1-1.html)
3. [http://www.cnblogs.com/xwang/p/3547685.html](http://www.cnblogs.com/xwang/p/3547685.html)
