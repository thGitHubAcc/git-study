## tomcat 添加配置
```sh
catalina.bat 添加

set "JAVA_OPTS=%JAVA_OPTS% -Xloggc:%CATALINA_HOME%/bin/gclogs/web-gc-%date:~0,10%.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=20M -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCCause "

	windows下：Xloggc 时间 参数:%date:~0,10%，需要将系统时间格式改为yyyy-MM-dd
	Eclipse 启动配置 时间 参数:%t
	linux下：

```
cmd visualvm
