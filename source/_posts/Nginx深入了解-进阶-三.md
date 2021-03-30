---
title: Nginx深入了解-进阶(三)
date: 2018-08-11 09:44:36
tags: nginx
---

Nginx负载均衡(Load Balance，简称LB)是一种服务器或网络设备的集群技术。负载均衡将特定的业务(网络服务、网络流量等)分担给多个服务器或网络设备，从而提高了业务处理能力，保证了业务的高可用性。

<!-- more -->

- Nginx负载均衡示意图:

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/20190215114531.png)

Nginx负载均衡原理就是将所有客户端的请求通过proxy_pass代理转发到对应的一组后端服务器upstream server上。

- 配置语法:

```nginx
Syntax：upstream name {…}

Default：–

Context：http
```

- 配置实例

> 后端服务器组upstream server

```nginx
#server1 
server { 
    listen 8001; 
    server_name localhost; 
    access_log /var/logs/nginx/access.log main; 
    location / { 
        root /opt/app/code; 
        index index.html index.htm index.php; 
    }
    ... 
}
```

```nginx
#server2 
server { 
    listen 8002; 
    server_name localhost; 
    access_log /var/logs/nginx/access.log main; 
    location / { 
        root /opt/app/code; 
        index index.html index.htm index.php; 
    } 
    ... 
}
```

```nginx
#server3
server { 
    listen 8003; 
    server_name localhost; 
    access_log /var/logs/nginx/access.log main; 
    location / { 
        root /opt/app/code; 
        index index.html index.htm index.php; 
    } 
    ... 
}
```

> 负载均衡服务器main server

```nginx
#默认使用轮询机制 
upstream test { 
    server 114.249.225.223:8001 down; 
    server 114.249.225.223:8002 backup; 
    server 114.249.225.223:8003 max_fails=1 fail_timeout=30s; // 允许失败一次，超过30s则直接访问8002 
} 

#加权 
upstream test { 
    server 114.249.225.223:8001; 
    server 114.249.225.223:8002; 
    server 114.249.225.223:8003 weight=5; 
} 

#ip_hash 
upstream test { 
    ip_hash; 
    server 114.249.225.223:8001; 
    server 114.249.225.223:8002; 
    server 114.249.225.223:8003; 
} 

#url_hash 
upstream test { 
    hash $request_uri; 
    server 114.249.225.223:8001; 
    server 114.249.225.223:8002; 
    server 114.249.225.223:8003; 
} 

server { 
    listen 80; 
    server_name localhost www.mantis.me; 
    access_log /var/logs/nginx/access.log main; 
    location / { 
        proxy_pass http://test; 
        include proxy_params; 
    } 
    ... 
}
```

> 后端服务器在负载均衡调度中的状态:

|参数|说明|
|---|---|
|down|当前的server暂时不参与负载均衡|
|backup|预留的备份服务器|
|max_fails|允许请求失败的次数|
|fail_timeout|经过max_fails失败后，服务暂停的时间|
|max_conns|限制最大连接数|

> 调度算法:

|算法|说明|
|---|---|
|轮询(默认)|按时间顺序逐一分配到不同的后端服务器|
|加权轮询|weight越大，分配到的访问几率越高|
|ip_hash|每个请求按访问ip的hash结果分配，这样来自同一个ip的请求将固定访问到同一个后端服务器|
|url_hash|按照访问的url的hash结果来分配请求，是每个url定向到同一个后端服务器|
|least_conn|最少连接数，哪个机器连接数少就分发到哪个机器|
|hash关键数值|hash自定义的key|