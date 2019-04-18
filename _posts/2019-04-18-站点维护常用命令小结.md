---
layout:     post
title:      "站点维护常用命令小结"
subtitle:   "记录一下方便以后修改"
date:       2019-04-18 12:00:00
author:     "rain"
header-img: "img/post-bg-build-server.png"
catalog: true
tags:
    - 建站
    - git
    - jekyll
---
## 前言
我在云服务器上和本地都有一份博客的代码，用git维护，放在github上，隔了一段时间没用发现命令忘的差不多，经常要查看history或者手册温习，为方便自己以后维护网站，打算写一篇总结记录常用命令。

##### git相关
###### 状态查看
有时候隔了一段时间忘记现在是什么状态，通过一下命令查看当前状态，是否有未commit的代码
 
```
git status
```
###### 提交某个代码修改
在状态里看到了某个代码修改，可以提交修改

```
git commit exampleFolder/exampleFilename
#倘若只有一个文件修改，你也可以在commit后面输入Tab键补全

```

###### 同步代码
云服务器 本地 github都有一份博客代码，为了保持一致性，提交前一般pull一下抓取最新的副本，合并冲突
 
```
git pull
```

###### 代码上传到github上

```
git push -u origin master
```

##### jekyll相关
###### 关闭后台的jekyll
```
pkill -f jekyll
```
	
###### 启动jekyll 后台运行
```
jekyll serve -B
```

##### jekyll主题由[Hux Blog](https://github.com/Huxpro/huxpro.github.io)提供 
###### 网站根目录下常用的目录和文件
```
_post           #文章
_include/en.md  #英文自我介绍
_include/zh.md  #中文自我介绍
_config.yml     #网站主页基本配置
_site           #jekyll自动生成的缓冲文件
img             #图片
```

继续深究请看[这里](https://www.jekyll.com.cn/docs/structure/)


