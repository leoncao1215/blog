---
title: "构建自己的博客"
date: 2020-03-03T11:43:01+08:00
draft: false
toc: false
images:
tags:
  - blog
  - Hugo
  - Nginx
---

## 使用 Hugo 构建博客框架

#### 首先通过 Homebrew 安装 Hugo。

```bash
brew install hugo
```

#### 使用 Hugo 初始化博客。

```bash
hugo new site my-blog
```

#### 添加主题，此处以 Hello Friend NG 主题为例。

```bash
cd my-blog
git init
git submodule add https://github.com/rhazdon/hugo-theme-hello-friend-ng.git themes/hello-friend-ng
```

#### 修改配置文件

以`themes/hello-friend-ng/exampleSite/config.toml`为模板修改即可。

#### 创建新的文章。

```bash
hugo new posts/my-first-post.md
```

#### 运行预览。

```bash
hugo server -D
```

## 生成静态文件

确保`content/posts/`下至少有一篇文章的`draft`属性为false，否则将导致 `/posts`路径为`404`。

#### 创建新的 git 模块方便部署。

```bash
git submodule add https://github.com/leoncao1215/blog-static.git public
```

#### 生成静态文件

```bash
hugo
```

## 使用 Nginx 部署到服务器

#### 在服务器上 clone 静态文件的项目。

```bash
cd /usr/share/nginx
git clone https://github.com/leoncao1215/blog-static.git my-blog
```

#### 配置 Nginx。

创建新的 nginx 模块化配置。

```bash
cd /etc/nginx/conf.d
vim blog.conf
```

在 blog.conf 中添加如下配置。若不用 HTTPS，将 443 端口的配置信息除 ssl 相关配置外全部放 80 端口的配置下即可。

```
server {
	listen 443 ssl;
	server_name c-leon.top www.c-leon.top;
	access_log /var/log/nginx/blog.log;
	root /usr/share/nginx/my-blog/;

	ssl_certificate cert/3237001_c-leon.top.pem;
        ssl_certificate_key cert/3237001_c-leon.top.key;
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
}
server {
	listen 80;
	server_name c-leon.top www.c-leon.top;
	return 301 https://c-leon.top$request_uri;
}

```

