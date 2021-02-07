---
layout: post
title: Docker 摸索
color: 888888
description: 整理出一条 Docker 包装应用的路线，踩坑
---

## 希望用 Docker 做到

1. 自动运行 Flask 作为服务器
2. 能够使用 crontab

## 构建 Dockerfile

``` Dockerfile
FROM ubuntu

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
        python3 \
        python3-pip \
        nginx \
        cron \
        nano && \
    apt-get autoremove -y && \
    apt-get clean -y && \
    pip3 install \
        flask \
        gunicorn \
        gevent

WORKDIR /var/www/html
COPY ./nginx.conf /etc/nginx/sites-available/default
COPY . .

CMD ["/bin/bash", "start.sh"]
```

以 ubuntu 为基础镜像，使用 nginx gunicorn gevent flask 构建服务器。  
/nginx.conf 把 flask 的 5000 端口映射为 80  
复制完代码到 Docker，执行 start.sh，内容如下：  

``` sh
#!/bin/bash
cron
nginx
cd /var/www/html
crontab crontab.conf
gunicorn Server:app -c gunicorn.conf.py
```

原始 ubuntu Docker 啥也没有，也不开机运行软件。所以让 Docker 每次启动时执行 start.sh，依次启动 cron nginx，初始化 crontab，最后启动服务器。  
在 gunicorn 的配置文件中  

``` conf
workers = 5
worker_class = "gevent"
bind = "0.0.0.0:5000"
# daemon = True
```

gunicorn 目前没法设置为 daemon 运行，因为如果这样，Docker 执行到这会认为 gunicorn 执行完毕，然后就整个容器退出了。  

nginx.conf  

``` conf
server {
    listen 80;
    # listen 443 ssl;
    server_name 0.0.0.0;
    # ssl_certificate /etc/letsencrypt/live/xxxx/fullchain.pem;
    # ssl_certificate_key /etc/letsencrypt/live/xxxx/privkey.pem;
    root /var/www/html;
    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host \$host;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
    }
}
```

crontab.conf 里的内容就是通常的 crontab  
Server.py 就是通常的 Flask 应用  

## 跑起来

首先 run 一下，得到 container  
`docker run -d -p 80:80 damagecontrolstudio/xxxx`  
把这条加入服务器的 crontab（可不是 Docker 内）  
`docker container start xxxxxxxxxx`

## 此外

Docker 可以通过 hub.docker.com 自动构建；  
Docker 里面的内容维护起来似乎不大方便，因为修改之后要 build 要 push 还要 pull；  
为了加一块 10M+ 大小的应用，包装成 500M 的 Docker，值得吗？  
以上只是整理出一条 Docker 包装应用的路线，踩坑。  
