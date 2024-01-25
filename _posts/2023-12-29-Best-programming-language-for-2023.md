---
layout:     post
title:      "最好的编程语言"
subtitle:   
date:       2023-12-29 21:00:00
author:     "rain"
header-img: 
catalog: true
tags:
    - js
    - python
---


## 年度最佳编程语言感悟
经过今天这波操作，如果要选出一个最好的编程语言，
第一名我给js，虽然用python确实很快，但js随处都能跑

## 案发背景
年底了，公司各个部分在核对数据，一个管计费的部门，要求我们核对并对漏报的账单进行补报。
明明每天都是通过消息上报资源费用，总有一批集群大批量在某几天缺少上报信息，计费部门的老大又让我们在他们几百年不维护的破页面逐条点漏报。关键点了漏报，页面分页又会回到第一页，并且补报的资源还是提示漏报，只是点击漏报时提示成功，要等他们定时任务后面再刷新【相当智能的平台】

来几张图模拟这个过程：
选择了oracle之后会出现oracle本月需要补录的资源信息
![1](/img/Snipaste_2024-01-03_21-44-23.png)
点击详情之后，需要针对具体天点击补录，随后会提示上报成功
![2](/img/Snipaste_2024-01-03_21-44-36.png)

## 解决方案
点了几十次之后，觉得这个操作应该可以通过脚本实现，开始录制脚本

把页面操作分三步用脚本实现

第一步 获取补录资源列表 GET /under-reportings  
第二步 从列表得到具体资源id，请求资源详情 GET /under-reportings/{id}  
第三步 有了资源详情，筛选未上报的日期，组装请求体，对该资源进行上报 POST /auto-supplement  


## 代码实现（Python）
实际代码更复杂，以下代码简化了请求参数和细节
```
import requests

# 请求头页面抓，只列出部分
headers = {
    "cookie": "---",
    "x-csrf-token": "----"
}
# 第一步 获取补录资源列表 GET /under-reportings
under_reporting_response = requests.get("https://EXAMPLE/api/under-reportings", headers=headers, verify=False)
under_reporting_list = under_reporting_response['data']
for under_reporting in under_reporting_list:
    # 第二步 从列表得到具体资源id，请求资源详情 GET /under-reportings/{id}
    under_reporting_detail_response = requests.get("https://EXAMPLE/api/under-reportings/" + under_reporting["id"],
                                                   headers=headers, verify=False)
    under_reporting_detail_list = under_reporting_detail_response['data']

    # 第三步 有了资源详情，筛选未上报的日期，组装请求体，对该资源进行上报 POST /auto-supplement
    for under_reporting_day in under_reporting_detail_list:
        if under_reporting_day['status'] == 3:
            auto_supplement_request_body = {
                "id": under_reporting["id"],
                "day": under_reporting_day['day'],
                "resource_type": under_reporting_day['resource_type']
            }
            requests.post("https://EXAMPLE/api/auto-supplement", json=auto_supplement_request_body, headers=headers,
                          verify=False)

```

## 故事仍未结束
以上代码虽然很简单，很快写完，测试环境没问题

可在生产环境，严格实行三面隔离（也就是租户端，管理平台，企业空间，实施逻辑隔离，管理平台须在堡垒机登录），这个管理平台仅能在堡垒机（即windows跳板机，数据拷贝拷出严格限制），跳板机里面可没有python环境

怎么办呢怎么办呢

## 故事结束
于是我想试试在部署服务的主机上复刻bash版本，前面都很顺利，curl命令请求也有相应，可无奈三个请求的相应解析，需要解对象里的数组，仅仅通过grep加正则，处理一个对象都要很久，奈何还有对象里再拆对象。

正当我准备放弃，脑壳灵机一动，何不试试写js，解析json正好时js最拿手活，于是乎我就在堡垒机打开记事本一顿手撸js。

堡垒机里有浏览器，写完f12丢console执行，经过一下午不屑努力，我终于用fetch方法复刻这个python代码。最后脚本跑完了。
