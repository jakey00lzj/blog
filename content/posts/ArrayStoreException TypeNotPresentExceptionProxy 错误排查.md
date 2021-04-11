+++
Description = "记录一下spring 组件版本升级带来的一些异常"
Tags = ['spring','error']
date = "2021-01-29T17:11:12+08:00"
title = "ArrayStoreException TypeNotPresentExceptionProxy 错误排查"
authors = ["jakey"]

+++

升级spring 组件版本后，应用起不来，出现 *ArrayStoreException TypeNotPresentExceptionProxy* 的异常

<!--more-->

异常如下图：

![Example image](/static/proxyerr1.png)



相关文档链接：https://blog.csdn.net/hengyunabc/article/details/79475505https://github.com/spring-projects/spring-boot/issues/17282

具体原因可能是一些class找不到的异常，但是异常堆栈日志又没打印，这时需要手动打断点，debug进去看cause原因，断点可以打在TypeNotPresentExceptionProxy 里面，如下图：

![Example image](/static/proxysource.png)


本次出现这个问题的原因的 有个 NacosConfigService  类找不到，但是我程序里面并不需要用到，而在配置类里面会用 @ConditionalOnClass 注解要判断它是否存在，如下

![Example image](/static/proxysource2.png)


看官方issue，里面说到 ，jvm加载的机制可能会导致 @ConditionalOnClass 和 @Bean 出现问题，可以单独起个新的配置类，把 @ConditionalOnClass 放在类注解上

![Example image](/static/githubissue1.png)

![Example image](/static/proxysource3.png)