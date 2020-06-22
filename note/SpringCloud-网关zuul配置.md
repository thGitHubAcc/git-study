1. 添加maven依赖
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
<!-- 安全认证 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

2. 配置 application.yml
```sh
server:
  port: 9001
spring:
  application:
    name: zuul
  security:
    user:
      name: root
      password: root
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:8081/eureka/
  instance:
    hostname: zuul
    instance-id: zuul

##配置负载均衡
#client:  ##服务名
#  ribbon:
#    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule #随机策略

ribbon:
  ReadTimeout: 3000
  ConnectTimeout: 3000
  eureka:
    enabled: false

# 路由端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      show-details: always
      enabled: true
    routes:
      enabled: true
# 配置指定微服务的访问路径
zuul:
  sensitive-headers: token
  routes:
    client-zuul:
      path: /client-zuul/**
      service-id: client-zuul-config
    service-zuul:
      path: /service-zuul/**
      service-id: service-zuul-config
  ignored-services:
    - client
    - service
# url 前缀
  prefix: /api
# 是否移除前缀
  strip-prefix: true
# 一下配置，表示忽略下面的值向微服务传播，以下配置为空表示：所有请求头都透传到后面微服务。
# 忽略正则
#  ignored-patterns:
#   - /client-*/**

client-zuul-config:
  ribbon:
    listOfServers: localhost:8082,localhost:8085
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule #随机策略
service-zuul-config:
  ribbon:
    listOfServers: localhost:8083

logging:
  level:
    com.netflix: debug
    org.springframework: DEBUG
```

3. 启动类添加注解
//该注解声明这是一个zuul代理，该代理使用Ribbon来定位注册到eureka server上的微服务，同时，整合了hystrix，实现了容错。
@EnableZuulProxy


4. 配置安全认证
```sh
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;

/**
 * 安全认证
 */
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 关闭csrf
        http.csrf().disable();
        // 表示所有的访问都必须认证，认证处理后才可以正常进行
        http.httpBasic().and().authorizeRequests().anyRequest().fullyAuthenticated();
        // 所有的rest服务一定要设置为无状态，以提升操作效率和性能
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
    }
}
```


5. 测试
访问 http://127.0.0.1:9001/client-zuul/hello 





# 过滤器端点
访问 http://localhost:9001/actuator/filters 可查看
```sh
error: [], 在其他阶段发生错误是，走此过滤器。
post: [], 在调用微服务执行后。可用于添加header，记录日志，将响应发给客户端。
pre: [], 在请求被路由之前调用，可利用这种过滤器实现身份验证。选择微服务，记录日志。
route: [] 在将请求路由到微服务调用，用于构建发送给微服务的请求，并用http clinet（或者ribbon）请求微服务。
```

## 自定义过滤器 

```sh
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;
import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
import org.springframework.stereotype.Component;

/**
 * 自定义过滤器 继承 ZuulFilter 类
 */
@Component
public class MyFilter extends ZuulFilter {

    /**
     * 过滤器类型
     * PRE_TYPE
     * POST_TYPE
     * ROUTE_TYPE
     * ERROR_TYPE
     * @return
     */
    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    /**
     * 过滤器执行的顺序，返回数字越小 越先执行。
     * http://localhost:9001/actuator/filters 查看过滤器
     * 默认过滤器中 ServletDetectionFilter 设置的order 为 -3  若要保证 自定义过滤器第一个执行，order 小于 -3 即可
     * @return
     */
    @Override
    public int filterOrder() {
        return 0;
    }

    /**
     * 是否需要进行 过滤 ，返回 true 才执行 下面的run 方法
     * @return
     */
    @Override
    public boolean shouldFilter() {
        return true;
    }

    /**
     * 过滤器 过滤的具体逻辑
     * @return
     * @throws ZuulException
     */
    @Override
    public Object run() throws ZuulException {

        //获取上下文（重要，贯穿 所有filter，包含所有参数）
        RequestContext requestContext = RequestContext.getCurrentContext();
        //获取认证信息
        String authorization = requestContext.getRequest().getHeader("Authorization");
        //获取请求uri
        String requestURI = requestContext.getRequest().getRequestURI();
        if(!"/api/client-zuul/hello".equals(requestURI)){
            requestContext.setSendZuulResponse(false);
            requestContext.setResponseStatusCode(444);
            //重中之重，这里一定要加要给Response设置CharacterEncoding编码为UTF-8
            requestContext.getResponse().setCharacterEncoding("UTF-8");
            requestContext.getResponse().setContentType("text/html;cahrset=UTF-8");
            requestContext.setResponseBody("请求非法");
        }
        return null;

    }
}
```

PS:若有多个过滤器，每个过滤器都会执行
shouldFilter() 只对当前过滤器是否执行做判断


# 接口容错
```sh
import org.springframework.cloud.netflix.zuul.filters.route.FallbackProvider;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.client.ClientHttpResponse;
import org.springframework.stereotype.Component;

import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStream;

@Component
public class MyFallBack implements FallbackProvider {

    /**
     * 表明为哪个微服务提供回退
     * 服务Id ，若需要所有服务调用都支持回退，返回null 或者 * 即可
     * （返回的服务id 为 配置文件中的 对应的 zuul: routes: service-id）
     */
    @Override
    public String getRoute() {
        // TODO Auto-generated method stub
        System.out.println("route");
        return "*";
        //return "service-zuul-config";
    }

    @Override
    public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
        System.out.println("cause:" + cause);
            return response(HttpStatus.BAD_GATEWAY);
    }

    private ClientHttpResponse response(final HttpStatus status) {
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return status;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return status.value();
            }

            @Override
            public String getStatusText() throws IOException {
                return status.getReasonPhrase();
            }

            @Override
            public void close() {
            }

            @Override
            public InputStream getBody() throws IOException {
                String msg = "{\"msg\":\"服务故障\",\"value\":\""+status.value()+"\",\"status phrase\":\""+getStatusText()+"\",\"status\":\""+status+"\"}";
                return new ByteArrayInputStream(msg.getBytes());
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON);
                return headers;
            }
        };
    }
}
```