---
title: Nginx深入了解-高级(五)
date: 2018-10-12 14:08:20
tags: nginx
---

使用Nginx与Lua开发。Lua是一个简洁、轻量、可扩展性的脚本语言；充分结合Nginx的并发处理epoll优势和Lua的轻量实现简单的功能且高并发的场景。

<!-- more -->

##### Lua安装

```bash
yum install lua
```

##### Lua运行

```bash
# 命令行 
Centos-Aliyun$ lua 
Lua 5.1.4 Copyright(C) 1994-2018 Lua.org,PUC-Rio 
> orint("hello world") 
hello world 
# 脚本 
Centos-Aliyun$ cat test.lua 
#!/usr/bin/lua 
print("hello world") 
Centos-Aliyun$ chmod a+rx ./test/lua 
Centos-Aliyun$ ./test.lua 
hello word
```

##### Lua基本语法

一，注释：- 行注释

– [[

块注释

–]]

二，变量

```lua
a = 'alo\n123"'
a = "alo\n123""
a = '\97lo\10\04923"'
a = [[alo
123"]]
```

布尔类型只有nil和false是false，数字0、’’ 空字符串(‘ \0’)都是true

lua中的变量如果没有特殊说明则都是全局变量，局部变量使用local定义。

三，while循环语句

```lua
sum = 0
num = 1
while num <= 100 do
    sum = sum + num
    num = num + 1
end
print("sum = ", sum)
```

Lua没有++或者+=操作。

四，for循环语句

```lua
sum = 0
for i =1 100 do
    sum = sum + i
end
```

五，if判断语法

```lua
if age == 40 and sex == "Male" then 
  print("Mans age than 40") 
elseif age > 60 and sex ~= "Fameale" then - ~=表示不等于 
  print("age than 60"); 
else local age = io.read() - 从终断读取信息 
  print("Your age is"..age) - ..表示字符串拼接 
end
```

> io库分别从stdin和stdout读写的read和write函数

##### Nginx + Lua环境搭建

- 安装LuaJIT

```bash
wget http://luajit.org/download/LuaJIT-2.0.2.tar.gz 
make install PREFIX=/usr/local/LuaJIT 
export LUAJIT_LIB=/usr/local/LuaJIT/lib 
export LUAJIT_INC=/usr/local/LuaJIT/include/luajit-2.0
```

- ngx_devel_kit和lua-nginx-module安装

```bash
cd /opt/download 
wget https://github.com/simpl/ngx_devel_kit/archive/v0.3.0.tar.gz 
wget https://github.com/openresty/lua-nginx-module/archive/v0.10.9rc7.tar.gz 
tar -zxvf v0.3.0.tar.gz 
tar -zxvf v0.10.9rc7.tar.gz
```

- 重新编译nginx

```bash
cd /opt/download 
wget http://nginx.org/download/nginx-1.12.1.tag.gz 
tar -zxvf nginx-1.12.1.tag.gz cd nginx-1.12.1 
./configure --prefix=/etc/nginx \ 
--sbin-path=/usr/sbin/nginx \ 
--modules-path=/usr/lib64/nginx/modules \ 
--conf-path=/etc/nginx/nginx.conf \ 
--error-log-path=/var/run/nginx/error.log \ 
--http-log-path=/var/log/nginx/access.log \ 
--pid-path=/var/run/nginx.pid \ 
--lock-path=/var/run/nginx.lock \ 
--http-client-body-temp-path=/var/cache/nginx/client_temp \ 
--http-proxy-temp-path=/var/cache/nginx/proxy_temp \ 
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi-temp \ 
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi-temp \ 
--http-scgi-temp-path=/var/cache/nginx/scgi-temp \ 
--user=nginx \ 
--group=nginx \ 
--with-compat \ 
--with-file-aio \ 
--with-threads \ 
--with-http_addition_module \ 
--with-http_auth_request_module \ 
--with-http_dav_module \ 
--with-http_flv_module \ 
--with-http_gunzip_module \ 
--with-http_gzip_static_module \ 
--with-http_mp4_module \ 
--with-http_random_index_module \ 
--with-http_realip_module \ 
--with-http_secure_link_module \ 
--with-http_slice_module \ 
--with-http_ssl_module \ 
--with-http_stub_status_module \ 
--with-http_sub_module \ 
--with-http_v2_module \ 
--with-mail \ 
--with-mail_ssl_module \ 
--with-stream \ 
--with-stream_realip_module \ 
--with-stream_ssl_module \ 
--with-stream_ssl_preread_module \ 
--with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' \ 
--with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie' \ 
--add-module=/opt/download/ngx_devel_kit-0.3.0 \ 
--add-module=/opt/download/lua-nginx-module-0.10.9rc7 
make && make install ## make -j 4 && make install
```

##### Nginx调用Lua模块指令

> Nginx的可插拔模块化加载执行共有11个处理阶段


|                指令                 | 描述                                  |
| :---------------------------------: | :------------------------------------ |
|     set_by_lua、set_by_lua_file     | 设置Nginx变量，可以实现复杂的赋值逻辑 |
|  access_by_lua、access_by_lua_file  | 请求访问阶段处理，用于访问控制        |
| content_by_lua、content_by_lua_file | 内容处理器，接受请求处理并输出响应    |


> Nginx Lua API


|         API          | 描述                                  |
| :---: | :---: |
|       nix.var        | nginx变量                             |
|  ngx.req.get_header  | 获取请求头                            |
| ngx.req.get_uri_args | 获取url请求参数                       |
|     ngx.redirect     | 重定向                                |
|      ngx.print       | 输出相应内容                          |
|       ngx.say        | 同ngx.print，但是会最后输出一个换行符 |
|      ngx.header      | 输出响应头                            |
|          …           |                                       |