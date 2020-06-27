1. 添加maven依赖
 <!-- 配置中心客户端：config-client -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

2-1.配置 bootstrap.yml(优先级比application.yml高)
```sh
 #注册中心-此时注册中去用于找到config-server
eureka:
  client:
    #设置服务注册中心的URL
    service-url:
      defaultZone: http://localhost:8081/eureka/

#应用名称，配置文件名，此时:congif-client-dev.yml
spring:
  application:
    name: config-client
  cloud:
    config:
      discovery:
        enabled: true
        # config server 的服务id
        service-id: config-server
      # 环境
      profile: dev #要使用的配置文件

```

3. 启动类添加注解
@EnableEurekaClient
