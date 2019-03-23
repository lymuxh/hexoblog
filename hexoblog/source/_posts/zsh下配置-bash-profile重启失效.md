---
title: zsh下配置.bash_profile重启失效
date: 2019-03-23 10:42:46
tags: [zsh , bash_profile ]
---

## zsh问题

Mac安装oh my zsh后安装的pod、nvm等功能后，重启终端pod和nvm命令找不到。

`zsh: command not found: pod`

## 原因

这是因为zsh其默认启动执行脚本变为了 ~/.zshrc

## 解决

修改 ～/.zshrc 文件，在其中添加：
`source ~/.bash_profile、~/.bashrc `
等脚本文件
