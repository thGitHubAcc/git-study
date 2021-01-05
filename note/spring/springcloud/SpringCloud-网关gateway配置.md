#
```
getway 使用netty容器 
引入的jar 包中 若有tomcat依赖 启动会出现异常。
可在带有tomcat jar包的依赖中加入：
 <!--
        1. webflux与webmvc不兼容，否则会项目启动不起来
        2. webflux使用Netty作为容器，如果使用tomcat，接口转发正常，但是会导致服务间的数据无法解析
            java.lang.ClassCastException: org.springframework.core.io.buffer.DefaultDataBufferFactory
            cannot be cast to org.springframework.core.io.buffer.NettyDataBufferFactory 
 -->
<exclusions>  
	<exclusion>
	    <groupId>org.springframework</groupId>
	    <artifactId>spring-webmvc</artifactId>
	</exclusion>
	<exclusion>
	    <groupId>org.springframework.bootk</groupId>
	    <artifactId>spring-boot-starter-tomcat</artifactId>
	</exclusion>
	<exclusion>
	    <groupId>org.apache.tomcat.embed</groupId>
	    <artifactId>tomcat-embed-core</artifactId>
	</exclusion>
	<exclusion>
	    <groupId>org.apache.tomcat.embed</groupId>
	    <artifactId>tomcat-embed-el</artifactId>
	</exclusion>
	<exclusion>
	    <groupId>org.apache.tomcat.embed</groupId>
	    <artifactId>tomcat-embed-websocket</artifactId>
	</exclusion>
</exclusions>
```