1. 添加maven依赖
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
<!-- eureka服务端 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
<!-- 监控端点 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

2. 配置 application.yml
```sh
spring:
  application:
    name: eureka #应用名

server:
  port: 8080 #访问端口

eureka:
  server: 
    #关闭自我保护
    enable-self-preservation: false
    #清理服务间隔时间，毫秒
    eviction-interval-timer-in-ms: 5000

  client: 
    #往注册中心注册注册
    register-with-eureka: false   
    #拉去注册列表
    fetch-registry: false  
    service-url:
    #注册中心地址
      defaultZone:  http://127.0.0.1:8081/eureka/      
    #健康检查-需要引入actuator  
    healthcheck:
      enabled: true 
      
  instance:
    #表示将自己的ip注册到EurekaServer上-不配置或false表示将操作系统的hostname注册到server
    prefer-ip-addres: true 
    id-address: 127.0.0.1
    hostname: eureka-server 
    #发送心跳给server的频率，每隔这个时间会主动心跳一
    lease-renewal-interval-in-seconds: 1
    #Server从收到client后，下一次收到心跳的间隔时间。超过这个时间没有接收到心跳EurekaServer就会将这个实例剔除
    lease-expiration-duration-in-seconds: 1 


```

3. 启动类添加注解
@EnableEurekaServer


4. 启动后，网址输入 http://127.0.0.1:8081 可查看服务注册信息


#eureka-server 生产环境优化
## 为什么加入pom依赖和@EnableEurekaServer注解 就会变为eureka-server
```
@EnableEurekaServer
和pom 依赖中的 org.springframework.cloud:spring-cloud-netflix-eureka-server:2.2.3.RELEASE下的spring.factories 

org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.springframework.cloud.netflix.eureka.server.EurekaServerAutoConfiguration

组成rureka-server
```
## 如何启动？
> 启动条件
```
EurekaServerAutoConfiguration.class 中 
@ConditionalOnBean(EurekaServerMarkerConfiguration.Marker.class) 当有这个Marker类实例的时候创建对象

@EnableEurekaServer注解下 EurekaServerMarkerConfiguration.class 创建了Marker类

```
> 怎么启动？
```
EurekaServerAutoConfiguration.class 类中引入了EurekaServerInitializerConfiguration.class
而EurekaServerInitializerConfiguration实现了SmartLifecycle
SmartLifecycle实现了Lifecycle

SmartLifecycle类中，isAutoStartup()返回true，则会自动启动start()方法
```

## 自我保护
```
EurekaServerInitializerConfiguration.class 类中的start()方法中断点
F7进入 到EurekaServerBootstrap.class中的initEurekaServerContext()方法
从this.registry.openForTraffic(this.applicationInfoManager, registryCount)进入到PeerAwareInstanceRegistryImpl.class中openForTraffic方法，走到  super.postInit() F7进入
AbstractInstanceRegistry.class的postInit()中，创建了new EvictionTask()对象
随后开启schedule定时执行evict(long additionalLeaseMs)方法，定期将没有心跳的服务剔除，达到阈值时会进行自我保护判断
```

## 不同数量服务的自我保护优化
```
服务少的情况下 不开自我保护
服务多的情况下 开启自我保护
```

## 快速下线
```
参数配置
eureka:
  server:
    #关闭自我保护
    enable-self-preservation: false
    #清理服务间隔时间，毫秒
    eviction-interval-timer-in-ms: 1000
    #自我保护阈值
    renewal-percent-threshold: 0.85
```

## 缓存
```
配置 关闭缓存
use-read-only-response-cache: false （默认开启）
开启状态下 调用服务 会先从缓存中获取服务
ResponseCacheImpl构造函数中
调用Timer定时器
new Date(((System.currentTimeMillis() / responseCacheUpdateIntervalMs) * responseCacheUpdateIntervalMs)
                            + responseCacheUpdateIntervalMs),
                    responseCacheUpdateIntervalMs);
每隔responseCacheUpdateIntervalMs 默认三十秒重新查询可用服务                   

配置重新查询可用服务时间
response-cache-update-interval-ms: 1000

```

## 
```
存储服务实例
ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>> registry
            = new ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>>(); 
```

## 
```
FilterRegistrationBean
eureka server 所有的操作都是通过http请求完成的


集群同步
PeerAwareInstanceRegistryImpl.register()
replicateToPeers(Action.Register, info.getAppName(), info.getId(), info, null, isReplication);
```

## CAP 
1. 三级缓存
2. 从其他peer拉取注册表。CAP中没有满足C（一致性）
3. 自我保护
4. 服务测算

## server
1. 接受注册
2. 接受心跳
3. 下线
4. 获取注册列表
5. 集群同步

## eureka sserver:
源码：
注册 心跳 下线 剔除 拉取注册表 集群同步


生产中 集群部署
1. 
register-with-eureka: true 
fetch-registry: true
instance: 
  prefer-ip-addres: true
2.  
defaultZone:ip换为域名

生产中 高可用
eureka:
  client: 
    region: fj
        availability-zones:
          fj: z2
        service-url:
          z1: http://127.0.0.1:7901/eureka/,http://127.0.0.1:7902/eureka/
          z2: http://127.0.0.1:7903/eureka/,http://127.0.0.1:7904/eureka/
        prefer-same-zone-eureka: true

