# RabbitMQ

## linux 安装

添加erlang 源至yum存储库
rpm -Uvh https://download.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

安装erlang
yum install erlang

导入RabbitMQ源
rpm -Uvh https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.8/rabbitmq-server-3.6.8-1.el7.noarch.rpm

安装RabbitMQ公共库秘钥
rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc

安装RabbitMQ
yum install rabbitmq-server-3.6.8-1.el7.noarch.rpm

service rabbitmq-server start

如果安装后web界面访问不了，只需要在安装目录bin下执行  /usr/lib/RabbitMQ
#rabbitmq-plugins enable rabbitmq_management


## 普通集群搭建
1. 配置hosts
vim /etc/hosts
将节点ip写入文件中
192.168.52.130 master
192.168.52.131 node1
192.168.52.132 node2

2. 启动rabbit 服务(三个节点)
systemctl start rabbit-server
或
cd /var/lib/rabbitmq/bin
rabbitmq-server -deched


3. 将三个节点服务器中的/var/lib/rabbitmq/.erlang.cookie文件内容改为一样（路径也可能时$HOME）

4. 搭建集群 
暂停当前节点rabbitmq服务（node1节点）
rabbitmqctl stop_app
加入集群（--ram 磁盘节点，不加为内存节点）
rabbitmqctl join_cluster --ram rabbit@master
启动rabbitmq服务
rabbitmqctl start_app

其余node节点重复以上操作。

执行完后，node1与node2也建立了联系


5. 创建用户
```
# 第一、添加mq用户并设置密码
rabbitmqctl add_user mq 123456

# 第二、设置mq用户为管理员
rabbitmqctl set_user_tags mq administrator

# 第三、设置mq用户的权限，指定允许访问的vhost以及write/read
rabbitmqctl set_permissions -p "/" mq ".*" ".*" ".*"

    Setting permissions for user "live" in vhost "/" ...
    ...done.

# 第四、查看vhost（/）允许哪些用户访问
rabbitmqctl list_permissions -p /
    Listing permissions in vhost "/" ...
    mq .* .* .*
    ...done.

# 第五、配置允许远程访问的用户，rabbitmq的guest用户默认不允许远程主机访问。
cat /etc/rabbitmq/rabbitmq.config 
    [
    {rabbit, [{tcp_listeners, [5672]}, {loopback_users, ["mq"]}]}
    ].
    # ps：主机1设置完以上这些之后，在集群内的机器都会同步此配置，但是/etc/rabbitmq/rabbitmq.config文件不会同步。

rabbitmqctl list_users
    Listing users ...
    mq  [administrator]
    ...done.
    最后，可以选择删除默认guest用户（密码也是guest）

rabbitmqctl delete_user guest
```

7. 镜像集群(普通集群基础上)
任一节点执行
^：为匹配符，只有一个^代表匹配所有，^zlh为匹配名称为zlh的exchanges或者queue。
ha-mode：为匹配类型，他分为3种模式：all-所有（所有的queue），exctly-部分（需配置ha-params参数，此参数为int类型比如3，众多集群中的随机3台机器），nodes-指定（需配置ha-params参数，此参数为数组类型比如["rabbit@node1","rabbit@node2"]这样指定为node1与node2这2台机器。）。

rabbitmqctl set_policy ha-all "^" '{"ha-mode":"all"}'

其他节点查看
rabbitmqctl list_policies

