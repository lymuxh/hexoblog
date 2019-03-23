---
title: 博客yilia主题集成gitment
date: 2019-03-23 10:47:04
tags: [hexo , yilia , gitment]
---

## 前言

hexo博客之前搭建时选择了yilia主题，因为平时比较懒，博客一直也没怎么更新，最近事情不是很多，就想重新整理以下博客，顺便集成以下评论和打赏功能。
看了国内几个第三方评论系统，如：多说，友言，畅言，不是关闭就是需要进行备案，看到有人提供gitmeng，还是基于GitHub的，就选择了Gitment。

## 升级博客主题

之前实用的是yilia主题3.0版本,看了以下最新的是4.0的版本虽然作者貌似已经不更新了，但是新版本主题默认支持gitment。

下载替换不多说，接下来看配置文件,根据自己账号内容进行配置就可以，评论配置区域集成了：1、多说；2、网易云跟帖；3、畅言；4、Disqus；5、Gitment,我们只需要开启Gitment就可以了。

根据配置需要做的就是创建一个存储评估的一个代码仓库和一个Github的Oauth的clientID和clientsecret。

<!-- more -->

### 注册Oauth Appliatin

[点击此处](https://github.com/settings/applications/new)来注册一个新的 OAuth Application。应用名称可以随意填写，主页地址可以写自己博客的域名地址（比如我的：https://lymuxh.github.io/）, callback URL（一般是评论页面对应的域名,可以填博客主页如 https://lymuxh.github.io/）。

点击生成对应的clientID和clientsecret。填入配置文件对应位置。

<font color=red> 如果主题没有集成Gitment模块可以参照下面gitment作者博客进行手动引入。</font>


### 初始化评论

页面发布后，你需要访问页面并使用你的 GitHub 账号登录（请确保你的账号是下面配置评论repo 的 owner），点击初始化按钮。

之后其他用户即可在该页面发表评论。


### yilia 支付头像等图片配置

在 yilia/source/创建assets目录，并把对应图片放入该目录，然后把相对路径（比如：/assets/img/alipay.jpg）配置到下面的配置文件里面。


```
# Header

menu:
  主页: /
  所有文章: /archives
  随笔: /tags/随笔/

# SubNav
subnav:
  github: https://github.com/lymuxh
  weibo: http://weibo.com/p/1005052386397721/home
  #rss: "#"
  #zhihu: "#"
  #qq: "812715937"
  #weixin: lymuxh
  #jianshu: "#"
  #douban: "#"
  #segmentfault: "#"
  #bilibili: "#"
  #acfun: "#"
  mail: "mailto:812715937@qq.com"
  #facebook: "#"
  #google: "#"
  #twitter: "#"
  #linkedin: "#"

rss: /atom.xml

# 是否需要修改 root 路径
# 如果您的网站存放在子目录中，例如 http://yoursite.com/blog，
# 请将您的 url 设为 http://yoursite.com/blog 并把 root 设为 /blog/。
root: /

# Content

# 文章太长，截断按钮文字
#excerpt_link: more
# 文章卡片右下角常驻链接，不需要请设置为false
show_all_link: '展开全文'
# 数学公式
mathjax: false
# 是否在新窗口打开链接
open_in_new: false

# 打赏
# 打赏type设定：0-关闭打赏； 1-文章对应的md文件里有reward:true属性，才有打赏； 2-所有文章均有打赏
reward_type: 2
# 打赏wording
reward_wording: '谢谢你请我吃糖果'
# 支付宝二维码图片地址，跟你设置头像的方式一样。比如：/assets/img/alipay.jpg
alipay: /assets/alipay.jpg
# 微信二维码图片地址
weixin: /assets/wechat.jpg

# 目录
# 目录设定：0-不显示目录； 1-文章对应的md文件里有toc:true属性，才有目录； 2-所有文章均显示目录
toc: 2
# 根据自己的习惯来设置，如果你的目录标题习惯有标号，置为true即可隐藏hexo重复的序号；否则置为false
toc_hide_index: true
# 目录为空时的提示
toc_empty_wording: '目录，不存在的…'

# 是否有快速回到顶部的按钮
top: true

# Miscellaneous
baidu_analytics: ''
google_analytics: ''
favicon: /favicon.png

#你的头像url
avatar: /assets/head.jpg

#是否开启分享
share_jia: true

#评论：1、多说；2、网易云跟帖；3、畅言；4、Disqus；5、Gitment
#不需要使用某项，直接设置值为false，或注释掉
#具体请参考wiki：https://github.com/litten/hexo-theme-yilia/wiki/

#1、多说
duoshuo: false

#2、网易云跟帖
wangyiyun: false

#3、畅言
changyan_appid: false
changyan_conf: false

#4、Disqus 在hexo根目录的config里也有disqus_shortname字段，优先使用yilia的
disqus: false

#5、Gitment
gitment_owner: xxx      #你的 GitHub ID
gitment_repo: ''          #存储评论的 repo
gitment_oauth: 
  client_id: ''           #client ID
  client_secret: ''       #client secret

# 样式定制 - 一般不需要修改，除非有很强的定制欲望…
style:
  # 头像上面的背景颜色
  header: '#4d4d4d'
  # 右滑板块背景
  slider: 'linear-gradient(200deg,#a0cfe4,#e8c37e)'

# slider的设置
slider:
  # 是否默认展开tags板块
  showTags: false

# 智能菜单
# 如不需要，将该对应项置为false
# 比如
#smart_menu:
#  friends: false
smart_menu:
  innerArchive: '所有文章'
  friends: '友链'
  aboutme: '关于我'

friends:
  #友情链接1: http://localhost:4000/
  #友情链接2: http://localhost:4000/
  #友情链接3: http://localhost:4000/
  #友情链接4: http://localhost:4000/
  #友情链接5: http://localhost:4000/
  #友情链接6: http://localhost:4000/

aboutme: 所谓成长<br><br>大概只是少了些恣意而为<br>多了些随遇而安
```

## yilia 主题修改


因为主题作者不维护了，所以主题的启动统计上报访问404错误。
删除 yilia/source-src/js/report.js 文件 和 yilia/source-src/js/main.js 里面对report的引用。

删除 yilia/souce/main.0cf68a.js 文件中 192对应的js方法和标号192的调用

定义处：

```
192:function(e,t,n){"use strict";function o(e){var t=new RegExp("(^|&)"+e+"=([^&]*)(&|$)","i"),n=window.location.search.substr(1).match(t);return null!=n?unescape(n[2]):null}var r=n(388);if(n(197),window.BJ_REPORT){BJ_REPORT.init({id:1}),BJ_REPORT.init({id:1,uin:window.location.origin,combo:0,delay:1e3,url:"//litten.me:9005/badjs/",ignore:[/Script error/i],random:1,repeat:5e5,onReport:function(e,t){},ext:{}});var i=window.location.host,a=top===window,u=!(/localhost/i.test(i)||/127.0.0.1/i.test(i)||/0.0.0.0/i.test(i));a&&u&&BJ_REPORT.report("yilia-"+window.location.host);var l=o("f"),c="yilia-from";l?(a&&BJ_REPORT.report("from-"+l),r.set(c,l)):document.referrer.indexOf(window.location.host)>=0?(l=r.get(c),l&&a&&BJ_REPORT.report("from-"+l)):r.remove(c)}e.exports={init:function(){}}},
```

删除调用处：

```
,n(192)
``` 


### share_jia微信分享报错

原因百度云盘生成二维码接口关闭了

修改yilia/layout/_partial/post/share.ejs 文件中

`
<img src="<%- 'qrcode' in locals ? qrcode(sUrl) : '//pan.baidu.com/share/qrcode?url=' + sUrl  %>" alt="微信分享二维码">
`

为

` <img src="<%- 'qrcode' in locals ? qrcode(sUrl) : '//api.qrserver.com/v1/create-qr-code/?size=150x150&data=' + sUrl  %>" alt="微信分享二维码">
`


### 设置首页点击所有文章右滑动出现文章目录

 1.请确保node版本大于6.2，输入node -v查看版本号，不符合的自己百度修改！
 2.在博客根目录（注意不是yilia根目录）执行以下命令：

` npm i hexo-generator-json-content --save`

3.在根目录_config.yml里添加配置：

```
jsonContent:
    meta: false
    pages: false
    posts:
      title: true
      date: true
      path: true
      text: false
      raw: false
      content: false
      slug: false
      updated: false
      comments: false
      link: false
      permalink: false
      excerpt: false
      categories: false
      tags: true
```

### 设置添加文字统计

在博客根目录（注意不是yilia根目录）执行以下命令：

` npm i hexo-wordcount --save`

在yilia/layout/_partial/left-col.ejs 文件header-menu组件下面添加

```
		<nav>
			总文章数 <%=site.posts.length%>
		</nav>
		<nav>
			总字数 <span class="post-count"><%= totalcount(site, '0,0.0a') %></span>
	  </nav>

```

在yilia/layout/_partial/article.ejs 文件中head组件下面添加

```
      <div align="center" class="post-count">
        字数：<%= wordcount(post.content) %>字 | 预计阅读时长：<%= min2read(post.content) %>分钟
      </div>

```


[gitment作者博客](https://imsun.net/posts/gitment-introduction/)