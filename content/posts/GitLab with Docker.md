---
title: "GitLab With Docker"
date: 2020-03-08T10:45:49+08:00
draft: false
toc: true
images:
tags:
  - Docker
  - GitLab
---

## Install Docker

#### Uninstall old versions

```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

#### Set up the repository

Install requirements

```bash
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

Set up **stable** repository

```bash
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

#### Install docker engine

```bash
sudo yum install docker-ce docker-ce-cli containerd.io
```

#### Start Docker

```bash
sudo systemctl start docker
```



## GitLab image

#### Pull image

```bash
sudo docker pull gitlab/gitlab-ce
```

#### Run GitLab

```bash
sudo docker run --detach \
  --hostname gitlab.example.com \
  --publish 8443:443 --publish 8080:80 --publish 8022:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```



## Reverse Proxy

#### Install Nginx

Add Nginx repository

```bash
sudo yum install epel-release
```

Install nginx

```bash
sudo yum install nginx
```

Start nginx

```bash
sudo systemctl start nginx
```



#### /etc/nginx/conf.d/gitlab.conf

```
server {
	listen 80;
	server_name gitlab.example.com;

	location / {
		proxy_set_header HOST $host;
		proxy_set_header X-Forwarded-Proto $scheme;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_pass http://127.0.0.1:8080/;
	}
}
```

Open port

```bash
sudo firewall-cmd --permanent --zone=public --add-service=http 
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
sudo firewall-cmd --reload
```



## More...

[掘金 - 使用Docker搭建GitLab](https://juejin.im/post/5cc1df885188252d6c43fd91)