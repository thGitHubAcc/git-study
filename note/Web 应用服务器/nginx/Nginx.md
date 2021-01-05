## Nginx

### 作用
1. 反向代理
2. 负载均衡
3. HTTP服务器（动静分离）
4. 正向代理

1 反向代理
反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。简单来说就是真实的服务器不能直接被外部网络访问，所以需要一台代理服务器，而代理服务器能被外部网络访问的同时又跟真实服务器在同一个网络环境，当然也可能是同一台服务器，端口不同而已。
```
server {
        listen  80; # 监听端口
        server_name  localhost; #监听地址
        client_max_body_size  1024M;

        location / {
            proxy_pass  http://192.168.52.130:8021;  #请求转向的地址
            proxy_set_header Host $host:$server_port;
        }
}

```







#linux 安装nginx
nginx下载地址：https://nginx.org/download/

1. 解压
tar -zxvf nginx-1.9.9.tar.gz

##进入nginx目录
cd nginx-1.9.9
## 配置 
./configure --prefix=/usr/local/nginx 
```
无root权限时，更换路径
```

##执行make
make 
make install

##启动nginx 
cd /usr/local/nginx/sbin
./nginx //启动nginx



#前端vue程序配置
```
location / {
	try_files $uri $uri/ /index.html;
    root   /path/dist; ## dist文件夹路径
    index  index.html index.htm;
}
#后端端口请求的配置
location /xxyy {                 
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Nginx-Proxy true;
    proxy_set_header Connection "";
    proxy_pass http://127.0.0.1:3000/xxyy/;     #后台接口的代理路径
}
```