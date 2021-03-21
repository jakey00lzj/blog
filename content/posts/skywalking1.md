+++
Description = "记录一下使用skywalking过程中遇到的问题以及使用经验"
Tags = ['skywalking']
date = "2021-03-19T17:59:54+08:00"
title = "skywalking 使用经验总结"
authors = ["jakey"]
+++


经过一段时间的探索以及生产环境的部署集成，记录一下使用skywalking过程中遇到的问题以及使用经验。

<!--more-->

## 基本介绍

skywalking 是一款优秀APM追踪工具，特别是适合java微服务间的追踪信息，可以用来非常当前系统的性能情况，比如接口延时排名，接口追踪信息，CPM等等信息，可以以它作为工具用来发现一些慢的服务，列为系统优化的任务之一。

skywalking github 地址： https://github.com/apache/skywalking

skywalking document 地址： https://skywalking.apache.org/docs/

当前使用的skywalking版本是v8.3.0-v8.4.0(最近这段时间升级到8.4.0)


## 集成与部署

skywalking 总共分三个模块，oap server , ui 以及 skywalking agent 三部分，

oap server 和 ui 算是服务端，ui 比较简单，是网页端模块，oap server 负责中心计算服务

agent部分是挂在应用端（通过增加-javaagent:xxx.jar的启动参数) 随着应用端启动的



一般来说，还要配置一个数据存储引擎，可以ES，mysql，tidb 等等，当前选择ES存储（v6.8.13)

mysql 查询性能较差，不适合高频页面查询；tidb还未使用集成测试过，暂不评价。



### 服务端部署



























