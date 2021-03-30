---
title: Nginx深入了解-高级(四)
date: 2018-09-25 14:08:20
tags: nginx
---

Nginx的https原理及实际应用场景。

<!-- more -->

#### HTTPS加密协议原理：

##### 中间人伪造客户端和服务端:

> 使用CA证书解决中间人伪造客户端和服务端的风险，客户端对数字证书进行CA校验：

- 如果校验成功则利用公钥加密
- 如果校验失败则停止回话

##### HTTPS服务优化

- 方法一：激活keepalive长连接
- 方法二：设置ssl session缓存

配置实例:

```nginx
server { 
	listen 443; 
	server_name ....; 

	keepalive_time 100; // 设置超时时间 

	ssl on; 
	ssl_session_cache shared:SSL:10m; 
	ssl_session_timeout 10m; 
	
	ssl_certificate ....; 
	ssl certificate_key ....; 
	.... 
}
```