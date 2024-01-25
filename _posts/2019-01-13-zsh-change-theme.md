---
layout:     post
title:      "zsh更换主题"
subtitle:   "zsh的箭头提示符很简洁美观，可是我想换个主题了"
date:       2019-01-13 12:00:00
author:     "rain"
header-img: "img/post-2019-01-13-bg-iterm2-theme.png"
catalog: true
tags:
    - mac os
    - zsh
    - iterm2
    - git
---
## 实验环境

> iterm2 + zsh

## 前言
个人比较懒，用mac os大半年，装完系统和iterm2，设置zsh之后再也没有更改过终端的字体大小，主题。逛帖子的时候，发现了好看的主题，于是想动手试下。

## 开搞
##### 思路
> 改ZSH_THEME的值 ( -> 下载安装特定字体  -> 修改终端偏好设置的字体）-> 重启终端 

换主题要修改 ~/.zshrc 里面的ZSH_THEME的值，其次如果该主题依赖某个字体，我们就要安装之，并在终端（iterm2或自带terminal）的默认字体为该字体，重启终端即可。

我挺喜欢agnoster这个主题的。git下用得很舒服，有层次感。
[参考主题](https://github.com/robbyrussell/oh-my-zsh/wiki/themes)
##### 预览
![预览](/img/post-2019-01-13-bg-iterm2-preview.png)
