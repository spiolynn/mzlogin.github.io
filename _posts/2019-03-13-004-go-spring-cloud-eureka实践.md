---
layout:     post
title:      004-go-spring-cloud-eureka实践
subtitle:    "\"004-go-spring-cloud-eureka实践\""
date:       2019-03-13
author:     PZ
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - GoLang 
    - eureka 
---

## 008-go-eureka实践

### 1 搭建一个eureka 注册中心（server）

> https://www.jianshu.com/p/bde8e442ba0b

### 2 尝试spring boot 来一个client

> https://www.jianshu.com/p/c6e7aef059db


### 3 理解`Eureka`

> https://juejin.im/post/593cc4c25c497d006b876429

#### 1 What

> 服务注册等一系列功能

- 服务注册: Register

> 当Eureka客户端向Eureka Server注册时，它提供自身的元数据，比如IP地址、端口，运行状况指示符URL，主页等。

```
<?xml version="1.0" encoding="UTF-8"?>
<instance>
    <instanceId>vendor:e3e856dd-9191-437f-9134-0558efcc7ac5</instanceId>
    <hostName>127.0.0.1</hostName>
    <app>vendor</app>
    <ipAddr>127.0.0.1</ipAddr>
    <status>UP</status>
    <port enabled="true">8080</port>
    <securePort enabled="false">443</securePort>
    <countryId>1</countryId>
    <dataCenterInfo>
        <name>MyOwn</name>
    </dataCenterInfo>
    <metadata>
        <lancher>0</lancher>
        <management.port>8080</management.port>
    </metadata>
    <homePageUrl>http://127.0.0.1:8080/</homePageUrl>
    <statusPageUrl>http://127.0.0.1:8080/info</statusPageUrl>
    <healthCheckUrl>http://127.0.0.1:8080/health</healthCheckUrl>
    <vipAddress>vendor</vipAddress>
    <secureVipAddress>vendor</secureVipAddress>
</instance>
```

- Renew：服务续约

> Eureka客户会每隔30秒发送一次心跳来续约。 通过续约来告知Eureka Server该Eureka客户仍然存在，没有出现问题。 正常情况下，如果Eureka Server在90秒没有收到Eureka客户的续约，它会将实例从其注册表中删除。 建议不要更改续约间隔。

- Fetch Registries：获取注册列表信息

> Eureka客户端从服务器获取注册表信息，并将其缓存在本地。客户端会使用该信息查找其他服务，从而进行远程调用。该注册列表信息定期（每30秒钟）更新一次。每次返回注册列表信息可能与Eureka客户端的缓存信息不同， Eureka客户端自动处理。如果由于某种原因导致注册列表信息不能及时匹配，Eureka客户端则会重新获取整个注册表信息。 Eureka服务器缓存注册列表信息，整个注册表以及每个应用程序的信息进行了压缩，压缩内容和没有压缩的内容完全相同。Eureka客户端和Eureka 服务器可以使用JSON / XML格式进行通讯。在默认的情况下Eureka客户端使用压缩JSON格式来获取注册列表的信息。

- Cancel：服务下线

> Eureka客户端在程序关闭时向Eureka服务器发送取消请求。 发送请求后，该客户端实例信息将从服务器的实例注册表中删除。该下线请求不会自动完成，它需要调用以下内容：DiscoveryManager.getInstance().shutdownComponent()；

- Eviction 服务剔除

> 在默认的情况下，当Eureka客户端连续90秒没有向Eureka服务器发送服务续约，即心跳，Eureka服务器会将该服务实例从服务注册列表删除，即服务剔除。


#### 2 How

![image](https://user-gold-cdn.xitu.io/2017/6/11/398fdaf163f6e7101dee83b76e28ff36?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 角色
    - Eureka Serve
    - Eureka Client
        - Applicaton Service （服务提供者）
        - Application Client （服务消费者）

- 1 Eureka Client向Eureka Serve注册，并将自己的一些客户端信息发送Eureka Serve
- 2 Eureka Client通过向Eureka Serve发送心跳（每30秒）来续约服务的。 如果客户端持续不能续约，那么，它将在大约90秒内从服务器注册表中删除。
- 3 注册信息和续订被复制到集群中的Eureka Serve所有节点。 - 来自任何区域的EurekaClient都可以查找注册表信息（每30秒发生一次）。
- 4 根据这些注册表信息，Application Client可以远程调用Applicaton Service来消费服务。

> https://github.com/Netflix/eureka/wiki/Eureka-REST-operations <br> api接口

> ![image.png](https://upload-images.jianshu.io/upload_images/14744153-e84ddd83046b3a83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 3 go-微服务eureka client实践

> https://callistaenterprise.se/blogg/teknik/2016/05/27/building-a-microservice-with-golang/ <br> 实践





