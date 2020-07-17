配置mvn:
环境变量：MAVEN_HOME：安装路径
path:%MAVEN_HOME%\bin

windows:
选择Binary下载

2. 配置
2.1 系统环境变量配置
变量名：ROCKETMQ_HOME
变量值：安装路径

3. 启动
start mqnamesrv.cmd 启动NAMESERVER

start mqbroker.cmd -n 127.0.0.1:9876 autoCreateTopicEnable=true 启动BROKER

在conf\broker.conf 添加enablePropertyFilter=true开启sql过滤 启动命令  start mqbroker.cmd -n 127.0.0.1:9876 autoCreateTopicEnable=true -c ..\conf\broker.conf

4. RocketMQ插件部署
下载
地址：https://github.com/apache/rocketmq-externals.git

编译
rocketmq-externals\rocketmq-console\src\main\resources
application.properties
修改端口号和namesrvAddr地址
编译启动
用CMD进入\rocketmq-externals\rocketmq-console文件夹，执行mvn clean package -Dmaven.test.skip=true，编译生成。

编译成功之后，Cmd进入‘target’文件夹，执行‘java -jar rocketmq-console-ng-1.0.1.jar’，启动rocketmq-console-ng-1.0.1.jar。

3.测试
浏览器中输入‘127.0.0.1:配置端口’，成功后即可查看。
