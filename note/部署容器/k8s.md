# Kubernetes
容器编排工具
docker-compose docker官方单机容器编排工具
docker-swarm docker官方集群编排工具
kubernetes(k8s) 谷歌官方集群编排工具

职能
自动化容器的部署和复制
随时扩展或收缩容器规模
容器分组Group，并且提供容器间的负载均衡
实时监控，及时故障发现，自动替换

## 概念

1. 节点(node)
是k8s集群中相对于Master而言的工作主机。

2. pod (k8s 调度的最小单元)
容器的容器，可以包含多个Container
k8s最小可部署单元，一个pod就是一个进程
内部容器网络互通，每个pod都有独立虚拟ip
pod都是部署完整的应用或模块

3. label(标签)
label定义了pod,service,node,RC等对象的可识别属性，用来进行管理和选择。

4. Replication Controller(RC)
定义pod副本的数量。在Master内，Controller Manager进程通过RC的定义来完成Pod的创建、监控、启停等操作。
通过RC,kuberneters保证了集群中运行用户期望的副本数量

5. service(服务)
一组提供相同服务的pod的对外访问接口，service作用于哪些pod通过Label Selector决定。

6. Volume(存储卷)
volume是pod中能够被多个容器访问的共享目录。

7. Namespace(命名空间)
通过将系统内部的对象分配到不同的namespace中，形成逻辑上分组的不同项目、小组，便于不同的分组在共享使用整个集群的资源的同时还能被分别管理。

8. Annotation(注解)
用户任意定义的附加信息，便于外部工具进行查找。

## k8s安装 V1.14
安装途径
1. 使用kubeadmin通过离线镜像安装（推荐）
2. 使用阿里公有云平台k8s
3. 通过yum官方仓库安装(旧版本)
4. 二进制包的形式进行安装，kubeasz(github)

环境准备
```
1.  设置主机名与时区
timedatectl set-timezone Asia/Shanghai  #都要执行
hostnamectl set-hostname master   #132执行
hostnamectl set-hostname node1    #133执行
hostnamectl set-hostname node2    #137执行

2. 添加hosts网络主机配置,三台虚拟机都要设置
vim /etc/hosts
192.168.52.130 master
192.168.52.131 node1
192.168.52.132 node2

3. 关闭防火墙，三台虚拟机都要设置，生产环境跳过这一步
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
setenforce 0
systemctl disable firewalld
systemctl stop firewalld
```

### 通过kubeadmin安装

非离线包安装参考：https://www.blog-china.cn/blog/liuzaiqingshan/home/254/1591593688584

```
1. 将镜像包上传至服务器每个节点
mkdir /usr/local/k8s-install
cd /usr/local/k8s-install
XFTP上传安装文件

2. 按每个Centos上安装Docker
tar -zxvf docker-ce-18.09.tar.gz
cd docker 
yum localinstall -y *.rpm
systemctl start docker
systemctl enable docker

3. 确保从cgroups均在同一个从groupfs
#cgroups是control groups的简称，它为Linux内核提供了一种任务聚集和划分的机制，通过一组参数集合将一些任务组织成一个或多个子系统。   
#cgroups是实现IaaS虚拟化(kvm、lxc等)，PaaS容器沙箱(Docker等)的资源管理控制部分的底层基础。
#子系统是根据cgroup对任务的划分功能将任务按照一种指定的属性划分成的一个组，主要用来实现资源的控制。
#在cgroup中，划分成的任务组以层次结构的形式组织，多个子系统形成一个数据结构中类似多根树的结构。cgroup包含了多个孤立的子系统，每一个子系统代表单一的资源

docker info | grep cgroup 

如果不是groupfs,执行下列语句

cat << EOF > /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=cgroupfs"]
}
EOF
systemctl daemon-reload && systemctl restart docker

4. 安装kubeadm
# kubeadm是集群部署工具

cd /usr/local/k8s-install/kubernetes-1.14
tar -zxvf kube114-rpm.tar.gz
cd kube114-rpm
yum localinstall -y *.rpm

5. 关闭交换区
swapoff -a
vi /etc/fstab 
#swap一行注释

6. 配置网桥

cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

7. 通过镜像安装k8s

cd /usr/local/k8s-install/kubernetes-1.14
docker load -i k8s-114-images.tar.gz
docker load -i flannel-dashboard.tar.gz
```

### 构建集群
1. master主服务器配置
kubeadm init --kubernetes-version=v1.14.1 --pod-network-cidr=10.244.0.0/16
在执行完后， 依次执行以下三个命令
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

 // 该命令执行完kubeadm init后会打印在控制台上，由其他节点执行，跟随主节点
kubeadm join 192.168.52.130:6443 --token j1a37a.o54d1ktxu4v3m8e5 \ --discovery-token-ca-cert-hash sha256:4267b610fb6c3f4b6fd8c25aa6a967ff600972d0f60962b56816661a4fc5e57e 

```
获取节点
kubectl get nodes

查看所有命名空间的pod
kubectl get pod --all-namespaces

安装flannel网络组件(若 pod 都处于pending状态，安装网络组件后，状态为running)
kubectl create -f kube-flannel.yml
```

2. node节点服务器
执行
kubeadm join 192.168.52.130:6443 --token j1a37a.o54d1ktxu4v3m8e5 \
    --discovery-token-ca-cert-hash sha256:4267b610fb6c3f4b6fd8c25aa6a967ff600972d0f60962b56816661a4fc5e57e 
该命令若丢失，在master 上执行
kubeadm token list 
然后在node上运行
kubeadm join 192.168.52.130:6443 --token j1a37a.o54d1ktxu4v3m8e5 --discovery-token-unsafe-skip-ca-verification

启动服务
systemctl start kubelet
设置开机启动
systemctl enable kubelet

kubeadm/kubelet/kubectl
kubeadm 是kubernetes集群快速构建工作
kubelet运行在所有节点上，负责启动pod和容器，以系统服务形式出现
kubectl 是kebenetes命令行工具，提供指令

### 文件集群共享(基于NFS)
Neteotk File System 文件传输协议

下载安装NFS
yum install -y nfs-utils rpcbind
步骤
```
cd /usr/local/
mkdir data
cd data
mkdir www-data
cd www-data/

vim /etc/exports
/usr/local/data/www-data 192.168.52.130/24(rw,sync)

 #启动服务
systemctl start nfs.service
systemctl start rpcbind.service
 #开机启动
systemctl enable nfs.service
systemctl enable rpcbing.service

 #检查配置结果
exportfs

到其他节点中下载NFS
yum install -y nfs-utils
查看master 设置的共享文件夹
showmount -e 192.168.52.130
 # 映射资源文件夹 ip为master ip：共享文件夹路径  /mnt 映射到本节点哪个文件夹下 
mount 192.168.52.130:/usr/local/data/www-data /mnt
```

### 端口转发工具-Rinetd
是linux操作系统中为重定向传输控制协议工具
可将源ip端口数据转发至目标ip端口
在kubernetes中用于服务端口暴露

Rinetd 安装
```
cd /usr/local
wget http://www.boutell.com/rinetd/http/rinetd.tar.gz
tar -zxvf rinetd.tar.gz
cd rinetd
sed -i 's/65536/65535/g' rinetd.c #允许的端口范围
mkdir -p /usr/man/ # rinetd要求
yum install -y gcc # 安装c语言编译器
make && make install 

vim /etc/rinetd.conf
#0.0.0.0所有ip
# 所有ip的8000端口 会转发到指定的内部service的ip地址
0.0.0.0 8000 10.99.255.130 8000

rinetd -c /etc/rinetd.conf
```


### 命令
```
查看节点
kubectl get nodes

查看服务
kubectl get service

查看所有命名空间的pod
kubectl get pod --all-namespaces

初始化(master执行)
kubeadm init --kubernetes-version=v1.14.1 --pod-network-cidr=10.244.0.0/16

查看token
kubeadm token list

跟随master(节点执行)
kubeadm join 192.168.52.130:6443 --token [token] --discovery-token-unsafe-skip-ca-verification

# 创建部署|服务
kubectl create -f yml

# 更新部署配置
kubectl apply -f  

#查看已部署的pod
kubectl get pod [-o wide] 

#查看pod详细信息
kubectl describe pod pod名称

#查看pod输出日志  
kubectl logs [-f] pod名称 

#删除（pod也会删除）
kubectl delete deployment deployment名称

查看server ip
cat /etc/kubernetes/kubelet.conf | grep server
```


### 配置文件(tomcat为例)

#### 部署配置文件
1. 简化(不做任何其他的配置)
tomcat-deploy.yml
```
apiVersion: extensions/v1beta1
kind: Deployment # 类型
metadata:
  name: tomcat-deploy 
spec: 
  replicas: 2 #部署两个pod
  template:
    metadata: 
      labels: # 标签
        app: tomcat-cluster
    spec: 
      containers: 
      - name: tomcat-cluster 
        image: tomcat # 镜像来源
        ports: 
        - containerPort: 8080 

```

2. 部署配置挂载点,集群下资源文件共享
修改tomcat-deploy.yml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: tomcat-deploy
spec: 
  replicas: 2 #部署两个pod
  template:
    metadata: 
      labels: 
        app: tomcat-cluster
    spec: 
      volumes: #数据卷
      - name: web-app #别名
        hostPath:
          path: /mnt  # 宿主机原始目录
      containers: 
      - name: tomcat-cluster
        image: tomcat
        ports: 
        - containerPort: 8080
        volumeMounts: 
        - name: web-app
          mountPath: /usr/local/tomcat/webapps
```

在共享文件夹中创建test/index.jsp
输出当前IP地址
<%=request.getLocalAddr()%>

//192.168.52.130 为 master服务器地址
测试：浏览器中输入：192.168.52.130:8000/test/index.jsp 查看返回结果

3. 资源限制(使用的cpu和内存资源)
修改tomcat-deploy.yml
在containers下追加resources
```
      containers:
        resources:
          requests:
            cpu: 0.5
            memory: 200Mi
          limits:
            cpu: 1
            memory: 512Mi
```

#### 服务配置文件

1. 设置统一访问端口32500
tomcat-service.yml
```
apiVersion: v1
kind: Service
metadata: 
  name: tomcat-service
  labels: 
    app: tomcat-service
spec: 
  type: NodePort
  selector: 
    app: tomcat-cluster
  ports:
  - port: 8000
    targetPort: 8080
    nodePort: 32500 
```

1. 使用Rinted对外提供Service负载均衡支持
修改tomcat-service.yml配置文件
```
apiVersion: v1
kind: Service
metadata: 
  name: tomcat-service
  labels: 
    app: tomcat-service
spec: 
  selector: 
    app: tomcat-cluster
  ports:
  - port: 8000
    targetPort: 8080
```


### 部署web应用demo:
1. 构建共享资源文件夹(路径：/usr/local/app   /dist:jar包和配置文件，/sql：初始化数据库语句)
master主机:
编辑exports
vim /etc/exports
/usr/local/app/dist 192.168.52.130/24(rw,sync)
/usr/local/app/sql 192.168.52.130/24(rw,sync)

重启nfs
systemctl restart nfs.service
systemctl restart rpcbind.service

查看是否配置成功
exportfs

--- node节点
创建文件夹存放资源文件
mkdir /usr/local/app-dist
mkdir /usr/local/app-sql

映射资源文件
mount 192.168.52.130:/usr/local/app/dist /usr/local/app-dist
mount 192.168.52.130:/usr/local/app/sql /usr/local/app-sql

2. 部署数据库
2.1 sql-deploy.yml
```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: app-db-deploy
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: app-db-deploy
    spec:
      volumes:
      - name: app-db-volume
        hostPath:
          path: /usr/local/app-sql
      containers:
      - name: app-db-deploy
        image: mysql:5.7
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "root"
        volumeMounts:
        - name: app-db-volume
          mountPath: /docker-entrypoint-initdb.d
        - name: app-db-volume
          mountPath: /usr/local/app-sql
```
创建部署
kubectl creat -f sql-deploy.yml

检查数据库是否初始化完成

2.2 sql-service.yml
```
apiVersion: v1
kind: Service
metadata:
  name: app-db-service
  labels:
    app: app-db-service
spec:
  selector:
    app: app-db-deploy
  ports:
  - port: 3310
    targetPort: 3306
```

创建服务
kubectl creat -f sql-service.yml 


3. 部署项目
3.1 app-deploy.yml
```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: app-app-deploy
spec: 
  replicas: 2
  template: 
    metadata: 
      labels:
        app: app-deploy
    spec: 
      volumes:
      - name: app-volume
        hostPath: 
          path: /usr/local/app-dist # 宿主机中的目录
      containers:
      - name: app-deploy
        image: openjdk:8u222-jre
        # 容器部署完成后执行的命令
        command: ["/bin/bash"]
        args: ["-c","cd /usr/local/app-dist;java -jar app.jar"]
        volumeMounts:
        - name: app-volume
          mountPath: /usr/local/app-dist
```

创建部署
kubectl creat -f app-deploy.yml

3.2 app-service.yml
```
apiVersion: v1
kind: Service
metadata:
  name: app-service
  labels:
    app: app-service
spec:
  selector:
    app: app-deploy
  ports:
  - port: 9011
    targetPort: 9011

```

创建服务
kubectl create -f app-service.yml

application.yml中数据库连接ip地址用service-name代替

4. 使用rinetd暴露端口
vim /etc/rinetd.conf
0.0.0.0 80 [service ip] 9011

加载配置文件
rinetd -c /etc/rinetd.conf

Done.

### web ui dashboard 可视化界面
Master开启仪表盘
```
kubectl apply -f kubernetes-dashboard.yaml
kubectl apply -f admin-role.yaml
kubectl apply -f kubernetes-dashboard-admin.rbac.yaml
# 查看
kubectl -n kube-system get svc

http://192.168.52.130:32000 访问
```
dashbord 部署容器
部署->创建->输入相应的镜像信息



健康检查机制
LivenessProbe存活探针
用户判断容器是否存存活，若容器不包含LivenessProbe 则kubelet认为容器的LivenessProbe返回值永远成功

ReadinessProbe 可读性探针
用于判断容器是否着呢二公尺提供服务，即容器的ready是否为true

配置
exec:执行命令
httpGet: http请求检测
tcpSocket: 检查端口
timeoutSeconds: 超时时间
successThreshold:最少连续成功多少次才被认定为成功
failureThreshold：
periodSeconds:探测间隔
initialDelaySeconds: 初始化延迟到额时间

```
spec:
  containers:
    readinessProbe: 
      httpGet:
        path: /headthz
        port: 8080W
      initialDelaySeconds: 3
      periodSeconds: 3
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySconds: 15
      periodSeconds: 20
```

注意事项：
1. 生产环境中不要直接使用Pod，应该使用Deployment，StatefulSet等更高级的控制及。直接使用pod，当pod，当物理节点不可用时 也不会重新做调度
2. 配置健康检查。
3. pod中产生的数据最好只写入volume。
4. 做好pod的资源限制

集群外部服务发现 
traefix
nginx-controller,
kubernetes Ingresss Controller for Kong,
HAProxy Ingress controller

配置管理 ConfigMap
提供了向容器中注入配置信息的能力
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-conf
  namespace: axe-production
data:
  redis.conf:   

```



## 集群外部服务发现 ingress


## 配置管理（ConfigMap）

### 基于环境变量方式：
虽然环境变量读取 ConfigMap 很方便，但无法支撑 ConfigMap 动态更新

1. 创建configmap.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  special.how: very
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: env-config
  namespace: default
data:
  log_level: INFO
```
2. 创建
kubectl create -f configmap.yaml

3. 创建pod
```
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
    - name: test-container
      image: nginx
      command: [ "/bin/sh", "-c", "env" ]
      env:
        - name: SPECIAL_LEVEL_KEY    # 环境变量的名称
          valueFrom:                 # key "SPECIAL_LEVEL_KEY" 对应的值
            configMapKeyRef:
              name: special-config   # 环境变量取自 name=special-config的ConfigMap
              key: special.how       # configmap中的对应的值
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: env-config
              key: log_level
  restartPolicy: Never
```
kubectl create -f configmap-pod.yaml

查看日志输出可以看到打印的环境变量

--------------------------------------

### 基于volume方式
以 Volume 方式使用的 ConfigMap 支持动态更新。也就是说 ConfigMap 更新后，容器中的数据也会更新。

1. configmap.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap
data:
  config1: xxx
  config2: yyy
```
kubectl apply -f myconfigmap.yml

2. 创建容器
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: busybox
    name: app
    volumeMounts:
    - mountPath: /etc/foo         # 挂载到容器内的目录
      name: foo                   # 引用Volume的名称
      readOnly: true
    args:
    - /bin/sh
    - -c
    - sleep 10; touch /tmp/healthy; sleep 30000
  volumes:
  - name: foo                     # 定义Volume的名称
    configMap:                     
      name: myconfigmap           # 使用ConfigMap "myconfigmap"
```
Pod 创建以后可以看到 Kubernetes 会在指定的路径 /etc/foo 下为每条配置数据创建一个文件，文件名就是数据条目的 Key（这里是 /etc/foo/config1 和 /etc/foo/config2）。而 Value 则存放在文件中。
```
kubectl apply -f mypod.yml
kubectl exec -it mypod sh
ls /etc/foo
cat /etc/foo/config1
cat /etc/foo/config2
```

自定义存放数据的文件名
```
volumes:
  - name: foo
    configMap:
      name: myconfigmap
      items:
        - key: config1                   // configmap中的key
          path: my-group/my-config1      // 将该key对应的值挂载到该路径下
        - key: config2
          path: my-group/my-config2
```


## 配置管理（Secret ）
echo -n "root" | base64

secret有三种类型：
1. Service Account：用来访问Kubernetes API，由Kubernetes自动创建，并且会自动挂载到Pod的 /run/secrets/kubernetes.io/serviceaccount 目录中；

2. Opaque：base64编码格式的Secret，用来存储密码、密钥等；

3. kubernetes.io/dockerconfigjson ：用来存储私有docker registry的认证信息

```
apiVersion: v1
kind: Secret
metadata: 
  name: mysecret
type: Opaque
data: 
  username: cm9vdA==
  password: cm9vdA==     #root
```

#### 通过 Volume 方式
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: busybox
    name: app
    volumeMounts:
    - mountPath: /etc/foo
      name: foo
      readOnly: true
    args:
    - /bin/sh
    - -c
    - sleep 10; touch /tmp/healthy; sleep 30000
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```
kubectl apply -f mypod.yml
kubectl exec -it mypod sh
ls /etc/foo
cat /etc/foo/username
cat /etc/foo/password

自定义存放数据的文件名
```
volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
      - key: password
        path: my-group/my-password
```

#### 通过环境变量方式
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: busybox
    name: app
    args:
    - /bin/sh
    - -c
    - sleep 10; touch /tmp/healthy; sleep 30000
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
```
kubectl exec -it mypod sh
echo $SECRET_USERNAME


### Secret与ConfigMap对比
相同点：

key/value的形式
属于某个特定的namespace
可以导出到环境变量
可以通过目录/文件形式挂载(支持挂载所有key和部分key)
不同点：

Secret可以被ServerAccount关联(使用)
Secret可以存储register的鉴权信息，用在ImagePullSecret参数中，用于拉取私有仓库的镜像
Secret支持Base64加密
Secret分为kubernetes.io/Service Account，kubernetes.io/dockerconfigjson，Opaque三种类型,Configmap不区分类型
Secret文件存储在tmpfs文件系统中，Pod删除后Secret文件也会对应的删除。