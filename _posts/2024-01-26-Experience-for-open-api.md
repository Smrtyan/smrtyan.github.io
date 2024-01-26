---
layout:     post
title:      "open api 开发经验谈"
subtitle:   
date:       2024-01-26 21:00:00
author:     "rain"
header-img: 
catalog: true
tags:
    - open api
    - Java
---




## 背景
自从22年10月陆续向cf部门提供了齐套数据库集群操作api，对接过程，cf的人走了也一波又一波，也看了联调多次，谈谈跨部门提供open api一些经验，以警示



### 1. 接口url，请求体，返回体，统一格式
首先接口url用RESTful风格，真的让接口看起来更高级
就拿获取用户接口，不用RESTful前，有的人写 `GET /getUser` `GET /getUsers` `GET /getUserList` `GET /findUser` `GET /findUsers`
且不说名称加不加s，动词有人用get 有人用find，在外人看起来有点乱糟糟
> 而采用RESTful风格后，获取单个用户就是 `GET /users/{id}`，获取列表就是`GET /users`，创建用户则是`POST /users`，删除用户是`DELETE /users/{id}`

> 请求体的格式和返回体要有类似结构，具体格式要看公司规范（方便某些微服务中转，虽然刚开始觉得公司这条规定很傻，但是当大家都遵守这条规定，接口写得那叫不要太爽）


> 返回体的meta信息中带上时间，请求方法 url, 当接口报错时方便找日志

```
{
    "data": {

    },
    "meta": {
        "url": "/v1/xxx/api/getUser",
        "method": "GET",
        "time": "2023-12-19T08:07:01"
    }
}
```

### 2. 参数校验一定要做，报错要具体，接口文档要写细致
#### 首先就是参数校验，能做的参数一定要做到位
我负责的模块一个接口参数接近20个，由于前辈挖坑，很多参数都做在了前端，后端并没做出来，如果少参数直接调用程序，一律报错*系统错误，联系管理员* 导致报错时每次都要登录容器看具体请求参数分析报错发生原因，而绝大部份如果参数校验做好了，就不用每次人工看。提供齐套接口时，由于每个接口前辈都没怎么做参数校验，导致提供api时没啥参考，参数校验做的比较费劲，短时间开发还遗漏很多检查项，这坑在接口提供出去之后，发现一个填一个。

#### 其次，文档一定要写清楚
和自己部门前端对需求不同，自己前端对了一遍之后，前端页面基本不会改，而open api不同，每隔一段时间又会有新部门来校对，一个接口二十个参数，如果当初文档写得不清不楚，故障排除，一个接口的小小报错足够花掉你几个小时。基本上每次定位都是某个参数写错了，如果一开始文档备注清楚，就算对接人看不懂，自己看懂报错再核对接口文档，还是能很快分析道问题的。

### 3. 设计DELETE请求的接口切记，千万不要带请求体
虽然你使用DELETE请求，确实能够接收到请求体

当时某个接口失效，为了定位问题，群里拉了本部门+华为云同事接近20+人看了一整个下午，才发现是delete传的body导致接口报错

我们调用华为云接口前会经过内部路由出去，而就是出去的时候，不太清楚经过哪一层路由之后，DELETE请求的body无一例外都丢了

查阅网上资料也没有明令禁止，在stackoverflow的网友答复却看到这条定义


```
The 2014 update to the HTTP 1.1 specification (RFC 7231) explicitly permits an entity-body in a DELETE request:
```

> A payload within a DELETE request message has no defined semantics; sending a payload body on a DELETE request might cause some existing implementations to reject the request.

也许某些中间件转发请求时会按照这个不成文规定设计，因而DELETE请求的body可能在某些场景会丢失

### 4. 已经提供的接口，不要直接在原接口直接加校验
发生了一起生产事故，创建集群api，我们会调用网络组api查信息，结果他们在某个周末上线后，接口会对头信息（header中的某个对象）进行校验，导致接口500，创建集群报错不可用了。
不要在历史稳定的接口加了任何校验，如果确实存在某些风险（如安全问题），应该提前通知对方联调，或者告知对方旧接口（如Post /v1/xxx/users）还能提供x个月，让对方部门在我方接口下线前尽快切到（如Post /v2/xxx/users）

## 结束