---
layout:     post
title:      "一次问题定位和乐观锁运用"
subtitle:   
date:       2023-12-13 11:00:00
author:     "rain"
header-img: 
catalog: true
tags:
    - 乐观锁
---
平台购买页集成多个数据库类型，支持用户购买多种数据库，业务逻辑比较复杂，有很多参数上下联动 一项一项查下一个参数（实际上业务远比这个复杂，所以本文简化申请数据库只有以下选项）

数据库类型（MySQL，PostgreSQL，openGauss）

区域（欧洲纽波特，贵阳等，区域依赖数据库类型，例如openGauss支持站点比较少）

数据库版本（例如mysql有5.7和8.0，postgresql有11和12，openGauss有2.7，3.2，依赖区域和数据库，如有些区域只支持5.7的MySQL，有些区域只支持8.0）

推荐参数模板（每个数据库，需要根据类型，区域，版本确定，不同区域版本模板的id不同）

![image-20231213223506535](/img/image-20231213223506535.png)

## 出大事了

啥事了，当用户切换到openGauss数据库 购买后台报错：参数模板id不存在

回到前端页面一看猛然发现，参数模板咋是MySQL的。

![image-20231213223957471](/img/image-20231213223957471.png)





## 分析

刚开始以为是前台为参数在切换版本没设置空，因此切换过去一段时间内还是展示MySQL的参数。
但研究代码发现，切换版本有做清空，因此排斥了脏数据的原因。

而在不经意的多次切换数据库类型观察中，发现其实页面在切openGauss后有几秒时间是更新成openGauss正确的参数模板，如下图

![image-20231213224954361](/img/image-20231213224954361.png)

但瞬间被一股不知名魔法，又把参数才变成MySQL的参数

![image-20231213225152141](/img/image-20231213225152141.png)

于是开始分析网络请求，这个问题复现过程发现，是海外站点请求获取MySQL参数模板接口，一般比较慢，例如本文的海外MySQL需要3秒左右，才拿到响应值，前端这个操作是异步，因此在这个请求参数模板这个接口下发三秒内如果切换到openGauss，这时站点是国内，300毫秒左右接口返回，因此此时异步接口把参数模板更新成正确的openGauss参数模板

而当MySQL请求参数模板这个接口结束后，又将这个属性覆盖一次，因此前端展示了错误的参数模板
示意图如下
![image-20231213230806208](/img/image-20231213230806208.png)

## 解决方案



回忆MySQL有个乐观锁的原理，于是打算设置一个变量作为版本号，每次触发这个动作时，版本号加一，当异步接口请求结束，如果此时方法内版本号比全局版本号旧，则不再更新参数模板动作，仅当版本为最新时更新





## 代码实现

```java
import java.util.Random;
import java.util.concurrent.TimeUnit;

public class Main {
    /** 版本号*/
    static int version = 0;

    /** 参数模板 这里用字符串模拟 */
    static String config = null;

    public static void main(String[] args){
        // 模拟用户点击页面 触发了更新参数模板接口
        for (int i = 0; i < 10; i++) {
            final int nextVersion = ++version;
            // 模拟异步请求
            new Thread(()->{
                updateConfig(nextVersion);
            }).start();
        }
    }

    /**
     * 模拟请求获取参数模板
     *
     * @param currVersion 当前版本号
     */
    public static void updateConfig(int currVersion) {
        long l = System.currentTimeMillis();
        // 这里不是真是请求，按照时间戳生产一个参数模板名，仅做演示
        String currentConfig = "config_" + l + "_V_" + currVersion;
        try {
            // 模拟网络时延
            TimeUnit.SECONDS.sleep(new Random().nextInt(10));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 请求结束，仅当此次调用是最新调用才更新参数模板
        if (version == currVersion) {
            config = currentConfig;
        } else {
            System.out.println("Update failed. Version too low config:" + currentConfig + ",version: " + currVersion);
        }
        System.out.println("FETCH config:" + currentConfig + ",current version: " + currVersion);
        System.out.println("Actual Config:" + config + ",version: " + version);
        System.out.println();
    }

}
```

执行结果

```
Update failed. Version too low config:config_1702476364779_V_6,version: 6
FETCH config:config_1702476364779_V_6,version: 6
Actual Config:config_DEFAULT_V_0,version: 9

Update failed. Version too low config:config_1702476364779_V_7,version: 7
FETCH config:config_1702476364779_V_7,version: 7
Actual Config:config_DEFAULT_V_0,version: 10

Update failed. Version too low config:config_1702476364779_V_8,version: 8
FETCH config:config_1702476364779_V_8,version: 8
Actual Config:config_DEFAULT_V_0,version: 10

Update failed. Version too low config:config_1702476364779_V_3,version: 3
FETCH config:config_1702476364779_V_3,version: 3
Actual Config:config_DEFAULT_V_0,version: 10

Update failed. Version too low config:config_1702476364779_V_9,version: 9
FETCH config:config_1702476364779_V_9,version: 9
Actual Config:config_DEFAULT_V_0,version: 10

Update failed. Version too low config:config_1702476364779_V_5,version: 5
FETCH config:config_1702476364779_V_5,version: 5
Actual Config:config_DEFAULT_V_0,version: 10

Update failed. Version too low config:config_1702476364778_V_2,version: 2
FETCH config:config_1702476364778_V_2,version: 2
Actual Config:config_DEFAULT_V_0,version: 10

Update failed. Version too low config:config_1702476364778_V_1,version: 1
Update failed. Version too low config:config_1702476364779_V_4,version: 4
FETCH config:config_1702476364779_V_4,version: 4
FETCH config:config_1702476364778_V_1,version: 1
Actual Config:config_DEFAULT_V_0,version: 10

Actual Config:config_DEFAULT_V_0,version: 10

FETCH config:config_1702476364779_V_10,version: 10
Actual Config:config_1702476364779_V_10,version: 10
```



从结果看，仅当version是10时才更新了参数模板