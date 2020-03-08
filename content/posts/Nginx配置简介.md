---
title: "Nginx配置简介"
date: 2020-03-07T20:21:34+08:00
draft: false
toc: false
images:
tags:
  - Nginx
  - depoly
  - server
---

## 常用配置

#### 静态资源

在部署前端项目时，经常会将项目build成静态的html & JavaScript & css等文件，此时可用nginx进行web服务的配置。

```nginx
server {
	listen			 3000;			# 该配置监听的端口
	server_name  localhost; # 请求的域名
	root /usr/share/nginx/zf_frontend/dist; # 资源根目录

	location / {
	  try_files $uri @fallback;
	}

	location @fallback {
	  rewrite .* /index.html break;
	}

	error_page   500 502 503 504  /50x.html;
	location = /50x.html {
    root   html;
  }
}
```



- `server_name`：若是同一端口端口有多个配置，会优先选择server_name一致的。如：端口80下同时有域名为 `api.example.com `和`www.example.com`两个配置，那么如果有请求`http://api.example.com/resources`就会被解析到前一个配置中。
- `try_file`：依次匹配后面的路径是否存在，例子中会将所有的本地没有静态文件的请求转给`@fallback`。
- `rewrite`：重写请求的路径，服务器接收到的将是rewrite之后的路径，例子中除静态资源外所有的请求将被转发给`/index.html`。



#### 反向代理

```nginx
server {
	listen			 80;
	server_name  api.example.com;

	location / {
	  proxy_pass http://127.0.0.1:8080;
	}
}

server {
  listen			 80;
  server_name	 www.example.com;
  
  location / {
    proxy_pass http://127.0.0.1:3000;
  }
}
```



#### 负载均衡

```nginx
upstream zf_frontend{
  server 127.0.0.1:3000 		 weight=5;
  server backup1.example.com weight=2;
  server backup2.example.com max_fails=3;
  server backup3.example.com fail_timeout=10s slow_start=30s;
  server backup4.example.com down;

  server backup5.example.com backup;
  server backup6.example.com backup;
}

server {
	listen			 80;
	server_name  localhost;

	location / {
	  proxy_pass https://zf_frontend;
    health_check;
	}
}
```



#### 动态缓存

```nginx
proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=cache_zone:10m max_size=10g inactive=60m use_temp_path=off;

map $request_method $purge_method {
    PURGE   1;
    default 0;
}

upstream backend {
  ...
}

server {
    ...
    location / {
        proxy_pass 					http://backend;
        proxy_cache 				cache_zone;
        proxy_cache_key 		$uri;
        proxy_cache_purge 	$purge_method;
        # 当分配的服务器出现50X 错误时分配另一台服务器
        proxy_next_upstream	error timeout invalid_header http_500 http_502 http_503 http_504
    }
}
```



