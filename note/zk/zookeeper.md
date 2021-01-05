1. 下载
```
官网：https://zookeeper.apache.org/releases.html
```

2. 安装
> 2.1 windows下安装
```
1. 解压

2. 修改conf/下的zoo_sample.cfg 复制一份 改名 zoo.cfg

3. 修改zoo.cfg中的 dataDir=zookeeper安装路径下的data文件夹(没有手动建一个)

4. zoof.cfg添加dataLogDir=zookeeper安装路径下的log文件夹

5. 启动程序



err:
a:找不到或无法加载主类 org.apache.zookeeper.server.quorum.QuorumPeerMain
q: 包下载错啦= =下载bin包


ps:参数说明
tickTime：这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。

initLimit：这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不是用户连接 Zookeeper 服务器的客户端，而是 Zookeeper 服务器集群中连接到 Leader 的 Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。当已经超过 10 个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 5*2000=10 秒

syncLimit：这个配置项标识 Leader 与 Follower 之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是 2*2000=4 秒

dataDir：顾名思义就是 Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。

clientPort：这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。
``` 

3. java使用
> 3.1 引入依赖
```xml


```