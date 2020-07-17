1. 添加maven依赖
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

<dependency>
    <groupId>com.netflix.hystrix</groupId>
    <artifactId>hystrix-core</artifactId>
    <version>1.5.12</version>
</dependency>

<!-- 引入feign依赖 ，用来实现接口伪装 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>

<!-- 引入hystrix依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>

2. 配置 application.yml
```sh
spring:
  application:
    name: client
eureka:
  instance:
    hostname: client
    prefer-ip-address: true
  client:
    service-url:
      defaultZone: http://127.0.0.1:8081/eureka/ # 注册中心地址
server:
#  port: 8085
   port: 8082
ribbon:
  eager-load: # 配置饥饿模式
    enabled: true
    clients:
      - client 
feign:
  client:
    config:
      default:
        logger-level: none # basic 只记录请求方法和URL以及响应状态代码和执行时间。
                           # headers 记录基本信息以及请求和响应标头
                           # full 记录请求和响应的头文件，正文和元数据
logging:
  level:
    com.fth.client: debug 
```    

3. 启动类添加注解 及 代码
```sh
@EnableCircuitBreaker//开启熔断
@EnableFeignClients//开启Feign
@EnableEurekaClient//eureka客户端方式启动


/**
 * RestTemplate携带认证信息
 * @return
 */
@Bean
@LoadBalanced #使用ribbon负载均衡
public RestTemplate restTemplate(){
     return new RestTemplateBuilder()
            .basicAuthentication("root", "root")  //请求头携带 Auth 认证信息 
            .build();
}

PS: RestTemplate 和 Feign 二者选一使用即可

```

4. 配置Feign
```sh
import feign.auth.BasicAuthRequestInterceptor;
import org.springframework.context.annotation.Bean;

public class FeignAuthConfiguration {

    /**
     * Feign请求带上auth认证信息
     * @return
     */
    @Bean
    public BasicAuthRequestInterceptor basicAuthRequestInterceptor() {
        return new BasicAuthRequestInterceptor("root", "root");
    }
}

```

4.1 配置熔断
```java
//name = 服务名
@FeignClient(name = "serverid",fallbackFactory = UserFeignFallback.class)
public interface UserFeign {

  //value=该服务对应的uri
  @RequestMapping(value = "uri",method = RequestMethod.POST)
  String test();
}

//降级策略
@Component
public class UserFeignFallback implements FallbackFactory<UserFeign> {
    @Override
    public UserFeign create(Throwable cause) {
        return new UserFeign() {
            @Override
            public String test(SysUser user) {
                System.out.println("Gateway invoke usercenter 'user/login' failed,couse by:{}",cause.getMessage());
                return null;
            }
        };
    }
}
```

5. 配置Ribbon
```sh
import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;
import org.springframework.beans.factory.annotation.Configurable;
import org.springframework.context.annotation.Bean;

@Configurable
public class RibbonConfig {

    @Bean
    public IRule ribbonRule(){
        return new RandomRule(); //随机策略
    }
}

```

6. controller代码

```sh
import com.fth.client.feign.ServiceFeignInterface;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.servlet.http.HttpServletRequest;
import java.util.List;

@RestController
public class InvokeController {
    @Autowired
    private RestTemplate restTemplate;

    @Autowired
    private ServiceFeignInterface serviceFeign;

    String url = "http://";
    String service = "service/";//服务名
    String uri = "instance-info";//uri

    /**
     * Feign 调用
     * @param serviceName
     * @return
     */
    @RequestMapping("invoke-service-by-feign")
    @ResponseBody
    public List invokeServiceByFeign(String serviceName){
        return serviceFeign.getServiceInstance(serviceName);
    }

  
    /**
     * postForEntity
     * @param serviceName
     * @return
     */
    @RequestMapping("invoke-service-by-postentity")
    @ResponseBody
    public List invokeServiceByPostEntity(String serviceName){
        HttpHeaders headers = new HttpHeaders();
        MultiValueMap<String, String> paramMap = new LinkedMultiValueMap<String, String>();
        paramMap.add("serviceName", serviceName);
        HttpEntity<MultiValueMap<String, String>> httpEntity = new HttpEntity<MultiValueMap<String, String>>(paramMap,headers);
        return restTemplate.postForEntity(url+service+uri, httpEntity, List.class).getBody();
    }
}

```

7. 端点监控 
  7.1 单服务监控

   > 添加 actuator 包

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

   > 添加 hystrix 包

    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>

   > 添加 dashboard 包

    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
    </dependency>

   > 启动类添加：
   ```sh
      注解:
      @EnableHystrixDashboard//开启可视化

      @Bean
      public ServletRegistrationBean getServlet(){
          HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
          ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
          registrationBean.setLoadOnStartup(1);
          registrationBean.addUrlMappings("/actuator/hystrix.stream");
          registrationBean.setName("HystrixMetricsStreamServlet");
          return registrationBean;
      }
  ```
   > 网址：http://localhost:8082/actuator/hystrix.stream

   7.2 集中可视化

    > 