---
title: CocoaPods
date: 2016-10-11 14:12:47
tags: [CocoaPods,IOS]
---
# CocoaPods 是什么

当你开发iOS应用时，会经常使用到很多第三方开源类库，比如JSONKit，AFNetWorking等等。可能某个类库又用到其他类库，所以要使用它，必须得另外下载其他类库，而其他类库又用到其他类库，“子子孙孙无穷尽也”，这也许是比较特殊的情况。总之就是，手动一个个去下载所需类库十分麻烦，尤其是不同版本类库依赖的不同版本类库。另外一种常见情况是，你项目中用到的类库有更新，你必须得重新下载新版本，重新加入到项目中，十分麻烦。如果能有什么工具能解决这些恼人的问题，那将“善莫大焉”。所以，你需要 CocoaPods。

CocoaPods应该是iOS最常用最有名的类库管理工具了，上述两个烦人的问题，通过cocoaPods，只需要一行命令就可以完全解决，当然前提是你必须正确设置它。重要的是，绝大部分有名的开源类库，都支持CocoaPods。所以，作为iOS程序员的我们，掌握CocoaPods的使用是必不可少的基本技能了。

 <!-- more -->

# 如何下载和安装 CocoaPods

在安装CocoaPods之前，首先要在本地安装好Ruby环境。Mac中带有安装好的Ruby环境，如果安装CocoaPods时显示版本过低，参考[mac下升级ruby环境版本](http://blog.csdn.net/archer_sc/article/details/52043305)

假如你在本地已经安装好Ruby环境，那么下载和安装CocoaPods将十分简单，只需要一行命令。在Terminator（也就是终端）中输入以下命令（注意，本文所有命令都是在终端中输入并运行的）

`sudo gem install cocoapods`

如果在终端中敲入这个命令之后，会发现半天没有任何反应。原因无他，因为那堵墙阻挡了cocoapods.org。
但是，我们可以用淘宝的Ruby镜像来访问cocoapods。按照下面的顺序在终端中敲入依次敲入命令：

```bash
$ gem sources --remove https://rubygems.org/

//等有反应之后再敲入以下命令

$ gem sources -a https://ruby.taobao.org/

```

为了验证你的Ruby镜像是并且仅是taobao，可以用以下命令查看：

`$ gem sources -l`

# 如何使用 CocoaPods

例如使用CocoaPods，在项目中导入AFNetworking类库

AFNetworking类库在GitHub地址是：[https://github.com/AFNetworking/AFNetworking](https://github.com/AFNetworking/AFNetworking)

为了确定AFNetworking是否支持CocoaPods，可以用CocoaPods的搜索功能验证一下。在终端中输入：

`$ pod search AFNetworking`

过几秒钟之后，你会在终端中看到关于AFNetworking类库的一些信息。这说明，AFNetworking是支持CocoaPods，所以我们可以利用CocoaPods将AFNetworking导入你的项目中。步骤如下：

1. 新建一个项目，名字CocoaPodTest,终端cd 到项目目录（注意：包含CocoaPodTest文件夹、CocoaPodTest.xcodeproj、CocoaPodTestTest的那个总目录）

2. 在该目录下建立podfile

 `vim Podfile`
  键盘输入 i，进入编辑模式，输入:

  platform :ios, '7.0'

  pod "AFNetworking", "~> 2.0"

  然后按Esc，并且输入“:”号进入vim命令模式，然后在冒号后边输入wq,回车后发现CocoaPodTest项目总目录中多一个Podfile文件

3. 终端进入项目目录，执行如下命令：

  `pod install `

4. 现在打开项目不是点击 CocoaPodTest.xodeproj了，而是点击 CocoaPodTest.xcworkspace

5. 现在，你就可以开始使用AFNetworking.h啦。可以稍微测试一下，在你的项目任意代码文件中输入：

 `#import <AFNetworking.h>` 或者
` #import "AFNetworking.h"`


# 常见问题

1. 如何导入多个第三方库

 这就需要修改Podfile了，就是用vim编辑的那个保存在项目根目录中的文件，修改完了Podfile文件，需要重新执行一次pod install命令。

 例如：

 platform :ios

 pod 'JSONKit',       '~> 1.4'

 pod 'AFNetworking',  '~> 2.0'


2. 项目存在多个Target的时候，需要配置Podfile文件来支持新增加的Target，否则只支持项目默认建立时生成的Target：

 a、如果新建一个Target，命名为Second，并且Second与Test两个Target所需要的第三方支持相同，也就是使用相同的Pods依赖库，则可以使用
link_with关键字：

 link_with 'Test', 'Second'

 platform :ios

 platform :ios, ‘9.0’

 pod 'AFNetworking', '~> 2.0'

 b、如果不同的Target需要不同的依赖库，则可以

 platform :ios

 target :'Test' do

 pod 'Reachability'

 pod 'SBJson'

 pod 'AFNetworking'

 end

 target :'Second' do

 pod 'OpenUDID'

 end

注：更新可用 `pod update`
更多使用方法参考 https://guides.cocoapods.org
