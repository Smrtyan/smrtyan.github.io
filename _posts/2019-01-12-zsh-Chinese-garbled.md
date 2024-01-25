---
layout:     post
title:      "zsh 中文乱码"
date:       2019-01-12 12:00:00
author:     "rain"
header-img: "img/post-bg-2019-01-12-chinese.jpg"
catalog: true
tags:
    - zsh
    - mac os
    - shell
---
##### 前记
由于喜欢zsh的便捷，于是服务端，mac os 下，zsh为默认的shell
中文乱码一直是命令行下让人头疼的问题，于是安装系统的时候语言是选择en_US
之前一直见中文就躲，尽量不在代码中出现中文注释防止乱码

##### 中文乱码和服务端客户端都有关
有天ssh远程连接局域网的机器时，发现命令行居然出现了中文，神奇，因为那台服务器装系统的时候，我选的是英文！
后来我在另一台电脑连上去，本来该出现中文的地方变成乱码了。
那么，中文显示应该和本地的机器也有关，那么配置的时候，服务端本地端都要配置。
之前我是一直以为是服务端问题，修改系统设置也只是修改服务端的。思路错了，折腾好久都没有成功。
照着这个新思路，搜关键词， <b> zsh 中文乱码 </b> 很快就找到解决方案

##### solution

在服务端和本地都修改 ~/.zshrc
编辑器编辑 ~/.zshrc
在文件内容末端添加：
{% highlight ruby %}
export LC_ALL=en_US.UTF-8  
export LANG=en_US.UTF-8
{% endhighlight %}
接着重启一下终端，或者输入
{% highlight ruby %}
source ~/.zshrc
{% endhighlight %}
