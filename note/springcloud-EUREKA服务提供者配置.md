1. 添加maven依赖
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
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
    name: service
eureka:
  instance:
    hostname: service
    prefer-ip-address: true
    id-address: 127.0.0.1
  client:
    service-url:
      defaultZone: http://127.0.0.1:8081/eureka/ #注册中心地址
server:
  port: 8083
```  
3. 启动类添加注解
@EnableEurekaClient


4. 启动后 可在 http://127.0.0.1:8081 查看服务是否加入到注册中心


简单测试接口：

```sh
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;
import java.util.List;

@RestController
public class ServiceInstanceController {

    /**
     * 服务发现
     */
    @Autowired
    private DiscoveryClient discoveryClient;

    @RequestMapping("instance-info")
    @ResponseBody
    public List getServiceInstance(String serviceName){
    	//返回服务元数据
        List<String> services = discoveryClient.getServices();
        services.forEach(System.out::println);
        return discoveryClient.getInstances(serviceName);
    }

    @RequestMapping("hello")
    @ResponseBody
    public String hello(HttpServletRequest request){
        int port = request.getServerPort();
        return "hello："+port;
    }
}
```


