---
layout:     post
title:      "打算更换博客系统"
subtitle:   "是时候拥抱git了"
date:       2019-01-11 12:00:00
author:     "rain"
header-img: "img/post-bg-2019-01-11-jekyll.jpg"
catalog: true
tags:
    - centos
    - jekyll
    - shell
    - blog
    - lnmp
---
##### 前因
看了下boss直聘的招聘，好像人家喜欢动手能力强，有博客，github上星星多的毕业生？

##### wordpress好像不适合我
几次用wordpress搭了博客，后来都是出于某种原因重装系统。加上自己我懒，几次都没有备份文章。每次搭了坚持不到两个月又不想写了。感觉在网页后台编辑文章很难受，1M的服务器带宽加上垃圾校园网网速可以卡到你怀疑人生。

##### 数据驱动的理解
最近在中国大学mooc看嵩天老师的pyhton课，他说的一个词让印象深刻：“数据驱动”。他举了个画图的例子，在一个写好的python代码中，提供不同的数据文件，就能画不同的图。

##### why Jekyll
思考了'数据驱动'这个词。嗯，也许Jekyll是一个不错的选择。

模版可以自定义，亦可以用别的。一篇文章就是一个markdown文件，就算以后只保留这些文件，直接记事本打开也比较好看。另一方面，用markdown写文章可以有结构层次。

jekyll不需要用到数据库，备份也只需要tarball一下网站的目录。（考虑到快毕业了，学生机过期了就得做迁移了）

加上自己还是比较喜欢命令行的黑窗和vim，就打算用jekyll 的同时，练习一下git的命令。
从开始接触linux就一直用centos（换过debian），吐槽最多的问题就是软件版本太低。
没办法，生产环境不能太追新，当然，有时万不得已的时候还是得折腾。

###### 遇到困难
在CentOS 7 安装jekyll时又遇到软件版本低的问题。
{% highlight ruby %}
➜ ~ gem install bundler jekyll
Fetching: bundler-2.0.1.gem (100%)
ERROR: Error installing bundler:
bundler requires Ruby version >= 2.3.0.
{% endhighlight %}
这不，提示ruby版本不够高。这个问题比较好解决，直接搜索关键词 centos 版本低 就很快找到想要的答案。

##### 反思
用linux不必刻意追求什么发行版本吧，不要遇到问题就重装，这样进步不快，大部分问题的解决办法都是 （找线索/错误）-》（关键词搜索）-》（尝试）。

##### 展望
我觉得，坚持写博客有以下好处：

- [x] 动手能力（将来可以自己给博客添加插件）
- [x] 对文章纲要的提炼能力
- [x] 版本迭代能力
- [x] 认识更多大牛
- [x] 找工作时博客就是简历，能力展示
- [x] 打发时间（比打游戏好吧）
- [x] 退休时的回忆录
- [x] ...

所以，你也打算创建自己的博客了吗？
