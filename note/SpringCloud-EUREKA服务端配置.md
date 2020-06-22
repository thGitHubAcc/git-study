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
