1. 添加maven依赖
<!-- 配置中心服务端：config-server -->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-config-server</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

2. 配置 application.yml(本地)
```sh
#服务端口
server:
  port: 6001

#应用名称及验证账号
spring:
  application:
    name: config-server
  profiles:
    active: native （启用本地配置文件）
  cloud:
    config:
      server:
        native:
          search-locations: classpath:/config #配置文件所在路径
        bootstrap: true
#注册中心
eureka:
  client:
    #设置服务注册中心的URL
    service-url:
      defaultZone: http://localhost:8081/eureka/
  instance:
    hostname: localhost
    instance-id: config-server
```

2. 配置 application.yml(GIT)
```sh
#服务端口
server:
  port: 6001

#应用名称及验证账号
spring:
  application:
    name: config-server

  cloud:
    config:
      server:
        bootstrap: true
        git:
          #https://github.com/yueyi2019/online-taxi-config-profile.git
          uri: https://github.com/thGitHubAcc/configcenter
          username:
          password:
          #默认是秒，因为git慢
          timeout: 15

#  rabbitmq:
#    host: localhost
#    port: 5672
#    username: guest
#    password: guest
#注册中心
eureka:
  client:
    #设置服务注册中心的URL
    service-url:
      defaultZone: http://localhost:8081/eureka/
  instance:
    hostname: localhost
    instance-id: config-server

#management:
#  endpoints:
#    web:
#      exposure:
#        #yml加双引号，properties不用加
#        include: "*"
```


```sh
git 获取配置规则：根据前缀匹配
/{name}-{profiles}.properties
/{name}-{profiles}.yml
/{name}-{profiles}.json
/{label}/{name}-{profiles}.yml

name 服务名称 (spring-application-name)
profile 环境名称，开发、测试、生产：dev qa prd
lable 仓库分支、默认master分支

匹配原则：从前缀开始。
```

3. 启动类添加注解
@EnableConfigServer