#解压
```
在redis文件路径下，cmd 命令 启动:
redis-server.exe redis.windows.conf

如果想方便的话，可以把 redis 的路径加到系统的环境变量里，这样就省得再输路径了，后面的那个 redis.windows.conf 可以省略，如果省略，会启用默认的。
```

#命令(另开cmd窗口)
```
redis-cli.exe -h 127.0.0.1 -p 6379

keys * 查询所有

```

#服务(windows)
```
配置 path redis 安装路径

cmd -> redis-server.exe --service-install redis.windows.conf --loglevel verbose 

服务中设置自动启动
```