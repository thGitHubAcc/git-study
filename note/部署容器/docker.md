### 物理机时代
```
部署慢
成本高
资源浪费
难于扩展和迁移
受制于硬件
```

### 虚拟化时代
```
多部署
资源池
资源隔离
容易扩展
VM需要安装操作系统
```

### 容器化时代
```
容器是app层面的隔离
虚拟化是物理资源层面的隔离
```

### 应用场景
```
标准化的迁移方式
统一的参数配置
自动化部署
应用集群监控
开发与运维之间的沟通桥梁
```

### Docker
```
开源的应用容器引擎，基于Go语言开发
容器时完全使用沙箱机制，容器开销极低
Docker具备一定虚拟化职能

持续部署与测试
可移植性
环境标准化和版本控制
隔离性
安全性
```

Docker是容器化平台，提供应用打包，部署与运行应用的容器化平台

### 镜像
```
镜像是文件，是只读的，提供了运行程序完整的软硬件资源，是应用程序的集装箱
```
### 容器
```
是镜像的实例，由Docker负责创建，容器之间彼此隔离
```

centos 运行完后就自动退出了 需要保持运行需要执行-t 并进入/bin/bash
docker run -d --name xxx -it centos /bin/bash

### 容器间通信
```
单向通信
--link 容器别名 建立连接 
docker run -d --name web --link database tomcat

双向通信 bridge网桥
dockers network ls

创建新的网桥
docker network create -d bridge my-bridge

建立容器与网桥连接
docker network connect my-bridge 容器别名
```

### 容器间共享数据
```
1. 设置 -v挂载宿主机目录
docker run --name 容器名 -v 宿主机路径：容器内挂载路径 镜像名
docker run --name t1 -v /usr/webapps:/usr/local/tomcat/webapps tomcat


2. 通过 volumes-from 共享容器内挂载点
创建共享容器
dockers create --name webpage -v /webapps:/tomcat/webapps tomcat /bin/true

共享容器挂载点
docker run --volumes-from webpage --name t1 -d tomcat

```


### 容器编排工具 docker-compose
```
Docker Compose 单机多容器部署工具
通过yml文件定义多容器如何部署
WIN/MAX默认提供Docker Compose，Linux需安装

安装：
1. sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

2. sudo chmod +x /usr/local/bin/docker-compose

查看安装版本
docker-compose -version

查看日志
docker-compose logs

关闭容器
docker-compose down 

部署wordpress开源博客
1. 创建wordpress目录 （cd /usr/）
mkdir wordpress

2. 创建docker-compose.yml 配置文件
vim docker-compose.yml

3. 解析执行
docker-compose up -d

http://192.168.52.128:8000/可进入博客首页


修改配置文件后，重启容器
docker-compose up -d --force-recreate
```
docker-compose.yml文件内容：

```yml
version: '3.3'
services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
       WORDPRESS_DB_NAME: wordpress
volumes:
    db_data: {}

```

中央仓库
https://hub.docker.com/


### linux 安装docker
```
下载安装工具
yum install -y yum-utils device-mapper-persistent-data lvm2
（下载失败时：
	查下自己的网卡：ip add
	进入目录network-scripts：
	cd /etc/sysconfig/network-scripts

	用vi编辑器编辑ifcfg-ens33：vi ifcfg-ens33
	将ONBOOT属性更改为yes
  service network restart
）


设置安装源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
-
使用速度更快的安装源
yum makecache fast

安装
yum -y install docker-ce

运行
service docker start
```

### 命令
```
查看版本号
docker version 

远程仓库拉取镜像（tags 版本号）
docker pull 镜像名:<tags>

创建容器，启动应用
docker run 镜像名:<tags>

端口映射 8000宿主机映射端口 8080虚拟机中容器201端口
docker run -p 8000:8080 镜像名

-d 后台启动容器
docker run -p 8000:8080 -d 镜像名

--name 容器别名

查看本地镜像
docker images 

查看运行中的容器
docker ps 

查看容器元数据
docker inspect 容器id

停止容器
docker stop 容器id

删除容器（-f 强制删除）
docker rm <-f> 容器id

删除镜像
docker rmi <-f> 镜像名:<tags>

在容器中执行命令 [-t] 采用交互方式执行命令 
exit 退出 
docker exec [-it] 容器id 命令 
docker exec [-it] 容器id /bin/bash


docker build -t 机构/镜像名:<tags> Dockerfile 目录 

导出镜像
将一个或多个图像保存到tar存档
docker save [镜像id]  > [镜像文件输出路径]

导入镜像
docker load < [镜像文件路径]

```

### Dockerfile 镜像描述文件
是一个包含用户组合镜像的命令的文本文档
Docker通过读取Dockerfile中的指令按布自动生成镜像
```
FROM 镜像名:<tag>    # 设置基准镜像


MAINTAINER fth # 镜像所属机构 说明信息
WORKDIR path # 切换工作目录 ~= cd 目标路径 目录不存在时会自动创建
ADD docker-web ./docker-web # add 复制 将 path1复制到path2
```

1. FROM
FROM centos 制作基准镜像
FROM scratch 不依赖任何基准镜像base image
FROM 镜像名:<tag> 依赖于tomcat镜像
尽量使用官方提供的Base Image

2. 描述性信息
MAINTAINER 
LABEL version 
LABEL description

3. 设置工作目录 WORKDIR
WORKDIR /usr/local
WORKRID /usr/local/newdir #自动创建
尽量使用绝对路径

4. 复制文件 ADD & COPY
ADD hello /  # 复制到根路径
ADD test.tar.gz / # 添加根目录并解压
ADD 除了复制 还具备添加远程文件功能 

5. 设置环境常量 ENV
ENV JAVA_HOME /usr/local/openjdk8
RUN ${JAVA_HOME}/bin/java -jar test.jar
尽量使用环境常量，提高系统维护性



RUN CMD ENTRYPOINT 

RUN 在构建镜像时执行命令，修改镜像内部的文件

CMD | ENTRYPOINT 在创建容器时执行命令，修改容器内部的文件

RUN yum install -y vim #Shell 命令格式

RUN ["yum","install","-y","vim"] #Exec 命令格式

ENTRYPOINT 
Dockerfile中只有最后一个ENTRYPOINT 会被执行
推荐使用Exec格式命令
一定会被运行

CMD 默认命令
用于设置默认执行的命令
如Dockerfile出现多个CMD 则只有最后一个被执行
如容器启动时附加指令，则CMD被忽略，不一定被执行
-- docker run fth/docker_run ls 
推荐使用Exec命令




```
1.复制app jar包及配置文件到服务器中

2.编写app Dockerfile 自定义镜像

3.编写db Dockerfile 初始化数据库
mysql 官方初始化路径：WORKDIR /docker-entrypoint-initdb.d

```

compose
```
1. vim docker-compose.yml （文件名固定）

2. docker-compose up -d (启动)



```

