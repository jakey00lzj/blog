+++
Description = "Sentinel 集群流控搭建"
Tags = ['sentinel','springcloud']
date = "2021-03-29T15:41:11+08:00"
title = "Sentinel 集群流控搭建"
authors = ["jakey"]

+++
Sentinel 集群流控搭建细节记录

<!--more-->




## 一、相关组件

相关组件应用有：  **sentinel-dashboard , token-server , apollo配置中心 , 集成sentinel的应用**

**sentinel-dashboard** ：sentinel 流控监控web管理界面

**token-server：** 负责sentinel集群的token下发（sentinel开启集群流控模式时会用到）

**apollo配置中心：**集成sentinel应用和token-server的sentinel相关配置会存在apollo配置中心上

**集成sentinel的应用：** 即包含sentinel相关包的应用

### 1.1 原理概述

1. token-client的DefaultClusterTokenClient#requestToken 基于原生netty
   发送至
   token-server的TokenServerHandler#channelRead，最终由DefaultTokenService#requestToken处理

2. token-server对限流核心处理在ClusterFlowChecker#acquireClusterToken，大致原理是根据自己基于滑窗实现的RequestLimiter计算目前资源对应的qps

3. netty发送与接收的数据类为ClusterRequest/ClusterResponse，token-client获取ClusterResponse后将其包装为TokenResult，其中status和remaining比较关键:
   a. remaining表示当前窗口内剩余可请求的数量；
   b. status的状态有很多，OK代表未限流，BLOCKED代表已限流；还有几种比如client与server交互异常，则会使token-client本次降级为本地限流，具体处理参考
   com.alibaba.csp.sentinel.slots.block.flow.FlowRuleChecker#applyTokenResult
   
   ![Example image](/images/sentinel-source1.png)
   
   总体来说无token实体概念的存在，主要是靠status来判定结果。

## 二、部署架构图

   ![Example image](/images/sentinel-network-topology.png)

## 三、配置信息

Apollo配置主要是配置在appid为token-server上面的，

需要如下几个命名空间： **ptjg.token-server , ptjg.sentinel-${appname}** 

**ptjg.token-server**：存放server相关的配置信息，如下：

   ![Example image](/images/sentinel-token-config.png)

cluster-server-namespace-set： 这里的namespace不是Apollo中的namespace概念，它指的是Sentinel中每个应用名，也就是sentinel-dashboard左边显示的应用名称（一般应用启动时可以用 -Dproject.name=xxxx ，来指定，建议app.properties中的appid一致）。因此，该cluster-server-namespace-set即标识TokenServer端需要监听的业务应用有哪些。

cluster-server-transport-config： 指定token-server启动的端口，该端口用于和client交互token信息。

token-server-cluster-map ：该配置用于维护多token-server与其client间的关系。它里面是json数组的格式，数组里面每一个对应一个token-server的配置。图中的json数组中只有一项，即只有一个token-server配置，该配置中， clientSet 指的是该token-server 负责的应用ip， @后面的端口号是sentinel端口号（应用配置里面可以用 *csp.sentinel.api.port* 配置）。后面的ip和port是指token-server的ip与端口，machineId是${token-server.ip}@${token-server.sentinel-api-port}。 **该配置除了token-server本身在监听，应用端也需要监听该配置的变化。**

**ptjg.sentinel-${project.name} ：** 这个下面**不需要自己配**，是dashboard 推送应用规则存储的地方，但是**如果有新增应用需要提前先建好该命名空间。** 

**注意：由于Apollo命名空间长度限制32位(注：新版本现在可以64位)，虽然我们一般推送project.name可以和app.id一致，但是有时候app.id太长，导致该命名空间名称长度超过限制而不能创建，这时project.name不能和app.id一致，要减少字符串长度使其符合长度限制。**


   ![Example image](/images/sentinel-app-config.png)


sentinel-flow-rules 是流控的配置名，出了这个还有：

sentinel-param-flow-rules：热点参数配置名

sentinel-degrade-rules：降级配置名

sentinel-authority-rules：授权配置名

sentinel-system-rules：系统配置名



## 四、上线注意

1.token-server-cluster-map 需要手动维护，有新应用上线需要手动配置上去，主要是增加clientSet里面的信息

2.一些特殊情况处理：

a. dashboard 挂了 ？  对应用无大影响，重启即可

b. token-server 挂了？ 修改token-server-cluster-map中的配置值

c. 应用某节点挂了？ 对token-server无影响，无须修改

d. 应用新增节点？   需要新增token-server-cluster-map中的clientSet