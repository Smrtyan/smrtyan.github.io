---
layout:     post
title:      "配置网络和防火墙小记"
subtitle:   "搞机搞机搞机...."
date:       2018-12-18 12:00:00
author:     "rain"
header-img: "img/post-bg-2018-12-18-network.png"
catalog: true
tags:
    - centos
    - shell
    - 服务器
    - firewalld
    - network
    - 多网口
---
##### get一台局域网服务器
今天给台服务器装系统，感觉服务器和台式也就那样，只是体积更大，可以接更多显卡和硬盘，内存，处理器。装系统过程和普通机子没啥太大区别。
<br/>
处理器一般，不过有十框框，内存32GB，硬盘1T。至少比树莓派强，还可以做内网网盘用，做测试机。
![cpu info](/img/post-bg-2018-12-18-network.png)
##### 双网口配置

给服务器装的CentOS7 minimal，如果只是日常使用就不需要图形化，将来要做别的在考虑装图形化界面。

网卡上有两个网口。一个口直接接内网的接口（方便远程），一个口接路由器（多了一层NAT）。

本来想着，Linux应该很智能，我要上网他就用走路由器，我要远程操控我就通过走内网口。现实总是和理想有差异，结果是上不了网。curl 上不了外网，yum无法下载软件。

问题关键在于：服务器所在网段是网络中心预留的，这里要手动配置ip，子网掩码和网关。现在有两个网口。
我感觉是dns配置出错。查了资料后，我打开/etc/resolv.conf。里面没内容。加了一条 
{% highlight ruby %}
nameserver 8.8.8.8
{% endhighlight %}
服务器能正常上网了。
##### 防火墙配置的怪事
然后我遇到的另一个问题就是firewall-cmd。开始看了下域public 作用网口 eth0 eth1,对外开放端口无。于是想着开放ss和ssh的端口，之后查询该端口是否开放，结果得到返回结果no。执行firewall-cmd –reload也是一样。后来通过systemctl工具重启firewalld服务之后才正常。有意思的是，这个过程我一直是连着ssh的。
<br/>
我猜新的系统在ssh连接的时候在防火墙关闭ssh端口不会马上生效，会一直保持连接，直到本次ssh连接断开？所以就是这个原因我修改端口reload也不会马上起作用？
##### 参考文章
[CentOS7使用firewalld打开关闭防火墙与端口-莫小安](https://www.cnblogs.com/moxiaoan/p/5683743.html)
