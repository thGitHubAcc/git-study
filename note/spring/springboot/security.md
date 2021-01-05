## spring security

依赖 
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
引入依赖后，启动打印
Using generated security password: ******
当有页面请求后，未登录用户会跳转到security的登陆页面，用户名：user
登录后可正常进行访问。

------------------------
设置默认用户

application.properties
```
spring.security.user.name=admin
spring.security.user.password=admin
spring.security.user.roles=admin
```
登录页面的用户密码为：admin

------------------------
设置权限（单节点）

helloController
```java
	@GetMapping("/admin/hello")
    @PreAuthorize("hasPermission('ROLE_admin','r')")
    public String adminHello(){
        return "admin-hello";
    }

    @GetMapping("/user/hello")
    @PreAuthorize("hasPermission('ROLE_user','c')")
    public String userHello(){
        return "user-hello";
    }
```

重写hasPermission方法逻辑
```java
@Component
public class CustomPermissionEvaluator implements PermissionEvaluator {
    @Override
    public boolean hasPermission(Authentication authentication, Object targetRole, Object targetPermission) {
        // 获得loadUserByUsername()方法的结果
        User user = (User)authentication.getPrincipal();
        // 获得loadUserByUsername()中注入的角色
        Collection<GrantedAuthority> authorities = user.getAuthorities();

       for (GrantedAuthority auth : authorities){
       	  //判断用户是否是该角色
           if(auth.getAuthority().equals(targetRole)){
               //判断是否拥有改权限，可以通过查库或者缓存等查询该角色拥有的权限。 

               return true;
           }
       }
       return false;
    }

    @Override
    public boolean hasPermission(Authentication authentication, Serializable serializable, String s, Object o) {
        return false;
    }
}
```

注入自定义的PermissionEvaluator
```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter  {

    /**
     * 注入自定义PermissionEvaluator
     */
    @Bean
    public DefaultWebSecurityExpressionHandler customWebSecurityExpressionHandler(){
        DefaultWebSecurityExpressionHandler handler = new DefaultWebSecurityExpressionHandler();
        handler.setPermissionEvaluator(new CustomPermissionEvaluator());
        return handler;
    }
}
```
测试 （当前用户仍是配置文件中的admin）
测试 /admin/hello和/user/hello接口

------------------------------


