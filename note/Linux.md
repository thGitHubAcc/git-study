you need to be root to perform this command
```
su 输入密码 获取root权限

```

切换到根目录
```
cd / 
```

jdk安装
```
1. 查询要安装jdk的版本
yum -y list java*

2. 下载安装
yum install -y java-1.8.0-openjdk.x86_64

查询jdk版本
java -version

3. vim命令打开/etc/profile   vim /etc/profile 

在文件中加入 

      JAVA_HOME=/usr/local/jdk1.7.0_71

      CLASSPATH=.:$JAVA_HOME/lib.tools.jar

      PATH=$JAVA_HOME/bin:$PATH

      export JAVA_HOME CLASSPATH PATH 

source /etc/profile 重新加载环境变量

注意yum安装的得用yum卸载，普通rpm安装的得用rpm卸载，当然yum安装的也可以使用rpm命令进行卸载，但是使用yum卸载比较方便，yum相当于一个一键卸载


```

jdk卸载
```
1、先输入java -version 查看是否安装了jdk

2、如果安装了，检查下安装的路径 which java（查看JDK的安装路径） 

3、卸载 rm -rf JDK地址（卸载JDK）  rm -rf /usr/java/jdk/jdk1.8.0_172/

4、vim命令编辑文件profile  vim /etc/profile

删除配置的环境变量，至此JDK卸载完毕

5、检查下自带的jdk

命令：

rpm -qa |grep java

rpm -qa |grep jdk

rpm -qa |grep gcj

如果没有输入信息表示没有安装。

如果安装可以使用rpm -qa | grep java | xargs rpm -e --nodeps 批量卸载所有带有Java的文件  这句命令的关键字是java
```
