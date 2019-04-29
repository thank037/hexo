---
title: Ribbon——超时与重试
date: 2019-04-26 18:57:04
categories: 微服务专题
tags:
  - 微服务
---

[TOC]

## 前言

在上篇[源码分析——客户端负载Netflix Ribbon](https://www.xiefayang.com/2019/04/23/%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E2%80%94%E2%80%94%E8%BD%AF%E8%B4%9F%E8%BD%BD%E5%AE%9E%E7%8E%B0Netflix%20Ribbon/)中提到了**重试**, 在Spring Cloud中各组件关于重试的概念还是很容易混淆
<!-- more -->

你会发现在Netflix OSS中: Eureka, Ribbon, Zuul, feign, etc... 虽然每个功能组件都很清晰, 能够单独使用

在Spring Cloud中看似也很清晰, 但实际上Spring Cloud的整合中会把多个组件配套组合起来, 以达到简化分布式开发的目的

例如在深入实践组件Feign时, 你是不是需要了解feign-hystrix, ribbon相关的东西呢? 

再例如超时, Zuul, Feign, Hystrix还是RestTemplate, 它们都有timeout相关的概念, retry机制也类似...

还好关于各组件中的超时和重试, 已经有人对其做了总结, 附上原文链接:
- [Spring Cloud各组件超时总结](http://www.itmuch.com/spring-cloud-sum/spring-cloud-timeout/)
- [Spring Cloud各组件重试总结](http://www.itmuch.com/spring-cloud-sum/spring-cloud-retry/)

> ^_^感谢这些分享的人

<br>

## **补充**

由于原文中的总结比较简单, 这里再对重试做一些补充

### **spring-retry**
在Ribbon的重试中, 除了配置项, 还需要加入[spring-retry](https://github.com/spring-projects/spring-retry)

- [官网说明](http://cloud.spring.io/spring-cloud-static/Finchley.SR2/single/spring-cloud.html#retrying-failed-requests)
> **Retrying Failed Requests**
> Spring Cloud Netflix offers a variety of ways to make HTTP requests. You can use a load balanced `RestTemplate`, Ribbon, or Feign. No matter how you choose to create your HTTP requests, there is always a chance that a request may fail. When a request fails, you may want to have the request be retried automatically. To do so when using Sping Cloud Netflix, you need to include [Spring Retry](https://github.com/spring-projects/spring-retry) on your application’s classpath. When Spring Retry is present, load-balanced `RestTemplates`, Feign, and Zuul automatically retry any failed requests (assuming your configuration allows doing so).

- pom依赖
```xml
<dependency>
	<groupId>org.springframework.retry</groupId>
	<artifactId>spring-retry</artifactId>
</dependency>
```

<br>

### **配置项 (int)**

文中提及了三个配置项参数: 
```yaml
ribbon:
  # 同一实例最大重试次数，不包括首次调用
  MaxAutoRetries: 1
  # 重试其他实例的最大重试次数，不包括首次所选的server
  MaxAutoRetriesNextServer: 2
  # 是否所有操作都进行重试
  OkToRetryOnAllOperations: true
```

在ribbon-core中可以找到对应的默认值
```java
public class DefaultClientConfigImpl implements IClientConfig {
	
	// MaxAutoRetriesNextServer (retryNextServer：重试下一实例)
	public static final int DEFAULT_MAX_AUTO_RETRIES_NEXT_SERVER = 1; 

	// MaxAutoRetries (retrySameServer：重试相同实例)
    public static final int DEFAULT_MAX_AUTO_RETRIES = 0;
	
	// OkToRetryOnAllOperations
	public static final Boolean DEFAULT_OK_TO_RETRY_ON_ALL_OPERATIONS = Boolean.FALSE;
}
```

这里要理解三个参数的意义, 可以这样推算出: 
- 当MaxAutoRetries=1, MaxAutoRetriesNextServer=2时: `RetryCount = (1+1) * (2+1) = 6次`
- 当MaxAutoRetries=0, MaxAutoRetriesNextServer=1时: `RetryCount = (0+1) * (1+1) = 2次`

也就是: `RetryCount = (maxAutoRetries + 1) * (maxAutoRetriesNextServer + 1)`

<br>

### 配置项 (bool)

上面三个参数中还有一个bool参数: OkToRetryOnAllOperations

文中的解释是是否所有操作都进行重试, 这里的操作是指什么? 我第一眼看不懂

细节尽在源码中: 

```java
@Override
public RequestSpecificRetryHandler getRequestSpecificRetryHandler(
		RibbonRequest request, IClientConfig requestConfig) {
	if (this.ribbon.isOkToRetryOnAllOperations()) {
		return new RequestSpecificRetryHandler(true, true, this.getRetryHandler(),
				requestConfig);
	}
	if (!request.toRequest().method().equals("GET")) {
		return new RequestSpecificRetryHandler(true, false, this.getRetryHandler(),
				requestConfig);
	}
	else {
		return new RequestSpecificRetryHandler(true, true, this.getRetryHandler(),
				requestConfig);
	}
}
```

这是源码中的一段逻辑, 可以看到`isOkToRetryOnAllOperations`在这里进行了两个分支判断
1. 判断`isOkToRetryOnAllOperations=true`时, 返回RetryHandler实现
2. 判断HTTP Method
	- GET: 返回RetryHandler实现
	- !GET: 返回RetryHandler实现, 不一样的是其中二个bool值为False

可以看到, 它们都会返回一个RetryHandler实现, 不同处在于传入的参数, 更准确的说是第二个bool值参数

来看下RequestSpecificRetryHandler构造函数中的几个参数 
- `bool okToRetryOnConnectErrors`: 字面意思是重试连接错误, 都为true
- `bool okToRetryOnAllErrors`: 字面意思是重试所有错误, 只有HTTP Method不是GET时为false
- etc...

理解这两个参数, 你只需要能够区分两种常见的Exception, 即:
- `java.net.SocketTimeoutException: Read timed out`: 这里的超时代表服务器请求超时
- `java.net.ConnectException: Connection refused`: 连接被拒绝, 也就是无法连接到服务器

最大的区别在于SocketTimeoutException是服务端已经接收到请求, 而客户端没有正常接收(例如超时了)

需要明白, 这里的重试是指目标服务不可达或无法正常响应, 而不是指返回一个4xx, 5xx的错误重试

> 注意: OkToRetryOnAllOperations那里的HTTP Method判断作用的是服务提供者的HTTP方法, 并非调用方本身

<br>

### **关于Open Feign**

文中提到了Feign的重试中, 虽然Feign和Ribbon充实是独立的, 如果都启用可能会让重试次数叠加

所以Spring Cloud在后来版本改为`feign.Retryer#NEVER_RETRY`, 即默认不开启Feign的重试, 转而可以使用Ribbon的重试配置即可

但是, 注意过Spring Cloud Finchley版的可能知道Feign的引入出现了变化, 变成了open-feign

重点有以下几点
- OpenFeign有自己的重试机制
- spring-retry对于OpenFeign是不起作用的
- 但是OpenFeign中读取的配置还是`ribbon.MaxAutoRetries`和`ribbon.MaxAutoRetriesNextServer`, 依然生效

详细及源码分析可参考: [Spring Cloud Finchley OpenFeign的重试配置相关的坑](https://blog.csdn.net/zhxdick/article/details/89490339)

<br>

## **使用建议**

重试确实能提高服务的容错和可用性, 但是使用不当不如不要使用

**1: Hystrix超时时间**

Hystrix的超时时间必须大于超时的时间: 这点很容易理解, 如果小于超时时间, 那就熔断了, 没有机会重试了

**2: 考虑服务幂等性**

 这点在判断OkToRetryOnAllOperations逻辑的源码那里也能看出来, 在启用重试时一定要考虑下游服务接口的幂等性

 概述为以下两点:
 - 如果你的服务API没有完全考虑幂等性, 建议不要把OkToRetryOnAllOperations设置为True
 - 其次, 即使设置为False, 如果下游服务的GET方法涉及资源操作, 无法满足幂等, 那么不建议启用重试机制