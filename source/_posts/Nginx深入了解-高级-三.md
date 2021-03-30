---
title: Nginx深入了解-高级(三)
date: 2018-08-30 14:08:20
tags: nginx
---

Nginx 高级模块的使用。secure_link_module模块、geoip_module模块。

<!-- more -->

#### 一，secure_link_module模块

1,限制并允许检查请求的连接的真实性以及保护资源免遭未经授权的访问。
2,限制链接的生效周期。

##### 配置语法

> Syntax：secure_link expression;
> Default：–
> Context：http、server、location


> Syntax：secure_link_md5 expression;
> Default：–
> Context：http、server、location

图示：

![图示](https://raw.githubusercontent.com/chunlintang/imgLib/master/20190215113404.png)

配置实例:

```nginx
server { 
	listen 80; 

	... 

	location / { 
		secure_link $arg_md5,$arg_expires; 
		secure_link_md5 "$secure_link_expires$uri test"; 
		if ($secure_link = "") { 
			return 403; 
		} 
		if ($secure_link = "0") { 
			return 410; 
		} 
	} 
}
```

#### 二，geoid_module模块

> 给予IP地址匹配MaxMind GeoIP二进制文件，读取IP所在地域信息。

http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz

http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz

http_geoip_mpdule使用场景

- 区别国内外作HTTP访问规则
- 区别国内城市地域作HTTP访问规则

配置实例：

```nginx
# 将下载模块load 

load_module "modules/ngx_http_geoip_module.so"; 
load_module "modules/ngx_stream_geoip_module.so"; 

user nginx; 
worker_process 1; 

....
```

```nginx
# 读取下载的maxmind文件 
geoip_country /etc/nginx/geoip/GeoIP.dat; 
geoip_city /etc/nginx/geoip/GeoLiteCity.dat; 

server { 
	listen 80; 
	server_name localhost; 

	location / { 
		if ($geoip_counter_code != CN) { 
			return 403; 
		} 
	} 

	location /myip { 
		default_type text/plain; 
		return 200 "$remote_addr $geoip_country_name $geoip_country_code $geoip_city"; 
	} 
}
```