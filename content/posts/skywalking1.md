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



同时还要配置一个数据存储引擎，可以ES，mysql，tidb 等等，当前选择ES存储（v7.11)

mysql 查询性能较差，不适合高频页面查询；tidb还未使用集成测试过，暂不评价。



### 服务端部署

#### 1. OAP Server 

##### 集群选择

oap server推荐集群部署，因为如果监控的应用一多，单节点很容易扛不住，或者也有可能由于各种意外原因挂掉。它的集群支持zookeeper,consul,nacos,k8s,ectd等，可在配置文件 ***config/application.yml*** 里面的进行对应配置

##### 数据存储引擎

支持h2,mysql,es6,es7,tidb,influxdb等，推荐使用es7，查询快，体验好。相关配置也是在 ***config/application.yml*** 里面的，根据实际配置情况进行配置即可。

##### 配置管理

支持apollo,zk,nacos,etcd等常见的配置管理中间件，推荐与其他线上应用使用的配置管理保持一致即可。

这里要注意不是所有的配置都可以放在配置管理里面，具体文档可以参见：

https://github.com/apache/skywalking/blob/master/docs/en/setup/backend/dynamic-config.md

##### 上报OAP自身服务状态

默认是不打开的，建议打开，这样可以在UI界面 **仪表盘 **板块下点击 **SelfObservability **观察oap server 各实例的运行情况。

打开的配置是在 ***config/application.yml*** 里面的配置*telemetry*的*selector*是*prometheus*即可，即如下：

```
telemetry:
  selector: ${SW_TELEMETRY:prometheus}
```

同时，修改 fetcher-prom-rules/self.yaml 里面的实例url, 一般是设置成当前局域网ip，如果有规划主机名与ip映射，用主机名也可以。示例如下：

```
metricsPath: /metrics
staticConfig:
  # targets will be labeled as "instance", xx.xx.xx.xx 换成当前ip
  targets:
    - url: http://xx.xx.xx.xx:1234
      sslCaFilePath:
  labels:
    service: oap-server
```

UI上的SelfObservability里面也会以这个url标识当前的oap server实例

#### 2. UI

配置 ***webapp/webapp.yml*** 里面的oapserver列表即可

### 应用集成

agent包可以放在服务器上的一个公共位置，供这台服务器上的各应用使用。

这里要注意的是，agent的配置文件(***agent路径/config/agent.config***)中的 *collector.backend_service* 要配置各个Oap Server 的地址，多个可以用逗号分隔。这里有两种配置方法，

1.ip法，也是官方文档默认的，具体就是类似  *ip1:port1, ip2:port2, ip3:port3*  ， 可能会遇到的问题就是以后的扩容问题，比如随着skywalking集成的应用越来越多，这时要扩充oap server的实例，比如从两台oap server扩到四台，那么agent这里的配置也要对应修改到四台的配置，修改完，应用还要**重启**后才能生效。

2.主机法，因为skywalking源码里面是按ip和端口用分号":"分隔解析的，所有主机法也要配置一个端口，我们一般要选择80。类似  *oapserver.loadbalance.hostname:80*  , 同时 nginx 上面还要配置它的负载均衡， 由于请求是grpc，所有，nginx版本至少要 1.13 版本之后，启动grpc模块。（可以参考： https://github.com/apache/skywalking/issues/4367 ）

其他的配置理论上可以不用改，运行时动态设置，举例如下：

```
java -Dskywalking.agent.service_name=应用名  -javaagent:skywalking-agent包解压路径/skywalking-agent.jar -Dskywalking.logging.file_name=应用名.log   -jar xxxx.jar
```





## 常见异常

一些相关链接：  https://my.oschina.net/osgit/blog/4558674

下面是一些遇过的问题，

1.**Grpc server thread pool is full, rejecting the task**

升级es版本到7.11后解决，本质原因是es 客户端6.x的bulk类请求的bug ， 出问题通过jstack可以发现死锁，相关的ES issue 链接： https://github.com/elastic/elasticsearch/issues/47599



















