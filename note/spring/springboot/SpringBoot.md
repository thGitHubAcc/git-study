#Springboot 项目搭建

##mavven工程下,pom.xml配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.0.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>
    <groupId>com</groupId>
    <artifactId>fth</artifactId>
    <version>0.0.1</version>
    <packaging>jar</packaging>
    <name>springboot</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
</project>
```

##启动类
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class);
    }
}
```

# 多配置文件
```sh
application.yaml

spring:
  profiles:
    active: prod #使用哪个配置文件
server:
  port: 8080 #使用的配置文件有相应的配置时，该配置不生效，若无，则使用该配置


application-dev.yaml

server:
  port: 8081


application-prod.yaml 

server:
  port: 8082  

```

# 启动banner
```sh
在resources下创建banner.txt，启动时的banner替换为文本内容
```

# 集成swagger2

## 导入maven依赖
```xml
<!-- swagger -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.4.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.4.0</version>
</dependency>
```

## 2.添加swagger配置类
```java
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;


@EnableSwagger2
@Configuration
@ConditionalOnProperty(name = "swagger.enable",havingValue = "true")
public class Swagger2 {

    // swagger2的配置文件，这里可以配置swagger2的一些基本的内容，比如扫描的包等等
    @bean
    public Docket createRestApi(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                // 为当前包路径
                .apis(RequestHandlerSelectors.basePackage("com.fth.controller")).paths(PathSelectors.any())
                .build();
    }

    // 构建 api文档的详细信息函数,注意这里的注解引用的是哪个
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                // 页面标题
                .title("Spring Boot 测试使用 Swagger2 构建RESTful API")
                // 创建人信息
                .contact(new Contact("FanTh",  "https://www.cnblogs.com/zs-notes/category/1258467.html",  "364870745@qq.com"))
                // 版本号
                .version("1.0")
                // 描述
                .description("API 描述")
                .build();
    }
}
```

## 添加注解
```java
@RestController
@Api("hellocontroller api")
public class HelloController {
    @Autowired
    Person person;
    @Autowired
    Goods goods;

    @PostMapping("hello")
    @ApiOperation(value = "say hello" ,notes = "emmmmm")
    public String hello(HttpServletRequest request){
        return "hello " +person.getName() + " " + goods.getGoodsName();
    }
}


常用注解： 
- @Api()用于类； 
表示标识这个类是swagger的资源 

- @ApiOperation()用于方法； 
表示一个http请求的操作 

- @ApiParam()用于方法，参数，字段说明； 
表示对参数的添加元数据（说明或是否必填等）

- @ApiModel()用于类 
表示对类进行说明，用于参数用实体类接收 

- @ApiModelProperty()用于方法，字段 
表示对model属性的说明或者数据操作更改 

- @ApiIgnore()用于类，方法，方法参数 
表示这个方法或者类被忽略 

- @ApiImplicitParam() 用于方法 
表示单独的请求参数 

- @ApiImplicitParams() 用于方法，包含多个 @ApiImplicitParam
```


http://localhost:8080/swagger-ui.html

#security(springcloud gateway webflux环境)

## 1.pom依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>5.1.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>5.1.5.RELEASE</version>
</dependency>
```
## 2.配置类
```java
package com.morewis.scm.gateway.config.security;

import com.morewis.scm.gateway.config.security.handle.*;
import com.morewis.scm.gateway.config.security.manage.AccountAuthentication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.access.expression.method.DefaultMethodSecurityExpressionHandler;
import org.springframework.security.authentication.ReactiveAuthenticationManager;
import org.springframework.security.authentication.ReactiveAuthenticationManagerAdapter;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.method.configuration.EnableReactiveMethodSecurity;
import org.springframework.security.config.annotation.web.reactive.EnableWebFluxSecurity;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.core.context.SecurityContext;
import org.springframework.security.core.session.SessionDestroyedEvent;
import org.springframework.security.core.session.SessionRegistry;
import org.springframework.security.core.session.SessionRegistryImpl;
import org.springframework.security.web.server.SecurityWebFilterChain;
import org.springframework.security.web.server.authentication.AuthenticationWebFilter;
import org.springframework.security.web.server.authentication.RedirectServerAuthenticationEntryPoint;
import org.springframework.security.web.server.csrf.CookieServerCsrfTokenRepository;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.reactive.UrlBasedCorsConfigurationSource;
import org.springframework.web.server.WebFilter;
import org.springframework.web.server.session.CookieWebSessionIdResolver;
import org.springframework.web.server.session.WebSessionIdResolver;
import org.springframework.web.util.pattern.PathPatternParser;

import javax.annotation.Resource;
import java.security.Permissions;
import java.util.Iterator;
import java.util.List;

/**
 * @author FTH
 * @Title spring Security配置安全控制中心
 * @Description
 * @Copyright: Copyright (c) 2020
 * @Company: morelean
 * @since 2020-6-30
 */
@Configuration
@EnableWebFluxSecurity
@EnableReactiveMethodSecurity
public class WebSecurityConfig {
    /**
     * 不需要进行认证的URL请求
     */
    private static final String[] excludedAuthPages = {
            "/rest/v1/usercenter/user/login.html"
    };
    //登录成功处理器
    @Resource
    private LoginSuccessHandler authenticationSuccessHandler;
    //登录失败处理器
    @Resource
    private LoginFaillHandler authenticationFaillHandler;
    //无权限的处理器
    @Resource
    private AcessDeniedHandler acessDeniedHandler;
    //认证失败处理器
    @Resource
    private LoginEntryPoint authenticationEntryPoint;
    //注销登录处理器
    @Resource
    private LogoutHandler accountLogoutHandler;
    //注销成功处理器
    @Resource
    private LogoutSuccessHandler accountLogoutSuccessHandler;

    @Bean
    public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
        //跨域配置
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource(new PathPatternParser());
        source.registerCorsConfiguration("/**", buildCorsConfiguration());

        //配置认证登录请求页面
        SecurityWebFilterChain chain = http.authorizeExchange().pathMatchers(excludedAuthPages).permitAll()
                .and().formLogin().loginPage("/rest/v1/usercenter/user/login")
                //添加认证成功的处理
                .authenticationSuccessHandler(authenticationSuccessHandler)
                //添加认证失败的处理
                .authenticationFailureHandler(authenticationFaillHandler)
                //失败或无权限的处理
                .and().exceptionHandling().accessDeniedHandler(acessDeniedHandler).authenticationEntryPoint(authenticationEntryPoint)
                //登录请求url
                .and().logout().logoutUrl("/rest/v1/usercenter/user/logout")
                //注销处理
                .logoutHandler(accountLogoutHandler)
                //注销成功处理
                .logoutSuccessHandler(accountLogoutSuccessHandler)
                .and()
                //OPTIONS请求放过(跨域情况下,会先进行一次OPTIONS的请求)
                .authorizeExchange().pathMatchers(HttpMethod.OPTIONS).permitAll()
                //其余请求需要认证后才能通过
                .anyExchange().authenticated()
                //默认httpbasic的配置
                .and().httpBasic()
                //跨域配置
                .and().cors().configurationSource(source)
                //csrf防范(disable已被关闭)
                .and().csrf().csrfTokenRepository(CookieServerCsrfTokenRepository.withHttpOnlyFalse()).disable()
                .build();

        //配置自定义登录认证逻辑
        Iterator<WebFilter>  weIterable = chain.getWebFilters().toIterable().iterator();
        while(weIterable.hasNext()) {
            WebFilter f = weIterable.next();
            if(f instanceof AuthenticationWebFilter) {
                AuthenticationWebFilter webFilter = (AuthenticationWebFilter) f;
                //将自定义的AuthenticationConverter添加到过滤器中
                webFilter.setServerAuthenticationConverter(new AuthenticationConverter());
            }
        }
        return chain;
    }

    @Bean
    public ReactiveAuthenticationManager reactiveAuthenticationManager() {
        System.out.println("manager");
        return new ReactiveAuthenticationManagerAdapter((authentication) -> {
            if (authentication instanceof AccountAuthentication) {
                AccountAuthentication gmAccountAuthentication = (AccountAuthentication) authentication;
                if (gmAccountAuthentication.getPrincipal() != null) {
                    authentication.setAuthenticated(true);
                    return authentication;
                } else {
                    return authentication;
                }
            } else {
                return authentication;
            }
        });
    }


    private CorsConfiguration buildCorsConfiguration() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*");
        corsConfiguration.addAllowedMethod("*");
        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.setMaxAge(7200L);
        corsConfiguration.setAllowCredentials(true);
        return corsConfiguration;
    }

}

```

## 数据库密码加密
1. pom.xml
druid依赖
```
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.1</version>
</dependency>
```

2. 到druid包路径下执行命令
```
java -cp druid-1.2.1.jar com.alibaba.druid.filter.config.ConfigTools you_password

获取加密后的密码及公钥
```

3. aplication.yml
```
spring:
  application:
    name: ibms
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:mysql://135.125.161.109:8901/hninvdb?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
    username: hninv
    password: UGDnXkcdOC0h7dc9QH9l6QBU/xsxlKE2nJOX2+EgsaanUev/LFhWHvelPWB1FWesQCdsbSiDQ+o7eFyYV+2hKA==
    druid:
      filter:
        config:
          enabled: true
      connect-properties:
        config.decrypt: true
        # 公钥
        config.decrypt.key: MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAL8lXiDIHHelf/GLz60UavjxSEOaFliBKZ8VNuEgrG9+xeQ9/bMZIM0HrOfsJ1UDGoxWUhUZgBSzVrdYv8FRhysCAwEAAQ==
    stat-view-servlet:
      login-username:
      login-password:
```