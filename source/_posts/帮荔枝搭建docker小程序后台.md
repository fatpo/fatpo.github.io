---
title: 帮荔枝搭建docker小程序后台
date: 2020-12-01 23:28:42
tags: docker
category: docker
---
# 技术选型
```dtd
荔枝：nginx + web 前端
肥婆：spring + mysql
```
# 环境选型
```dtd
腾讯云Centos7.2 + 腾讯云域名 + 腾讯云 ssl 证书
```
PS：
```dtd
请务必选好域名后就备案！
```
# 前端
```dtd
告诉荔枝，你把前端代码 dist.zip 通过 ftp 传到 /root/webdocker/webapp 文件夹下面即可
```
## nginx 配置 docker
当前路径：
```dtd
[root@VM-0-9-centos webdocker]# pwd
/root/webdocker
```
Dockerfile:
```dtd
[root@VM-0-9-centos webdocker]# cat Dockerfile
FROM nginx:latest
MAINTAINER fatpo

RUN rm /etc/nginx/conf.d/default.conf
ADD default.conf /etc/nginx/conf.d/

COPY webapp/  /var/www/frontend/
```
编写 default.conf： 
```dtd
[root@VM-0-9-centos webdocker]# cat /root/webdocker/default.conf
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /var/www/frontend/dist/;
        index  index.html index.htm;
    }

    location /cms/
    {
        proxy_set_header   Host             $host:8080;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   Cookies          $http_cookies;
        proxy_pass http://myapp:8080;
    }
}
```
顺手写个脚本，不用每次都敲一堆命令：
```dtd
[root@VM-0-9-centos webdocker]# cat restartWeb.sh
#!/bin/bash

# 重打个前端镜像
docker build -t ouyangbro/dalihaozaiweb:latest .

# 停止并删除旧的容器，否则不会拉新镜像
docker stop  dalihaozaiweb
docker rm dalihaozaiweb

# 跑新的容器，注意 --link 里面的 myapp 就是我们后端请求地址
docker run --name dalihaozaiweb -p 80:80 -d --link root_dalihaozai_1:myapp ouyangbro/dalihaozaiweb:latest

# 删除无用镜像
docker images|grep none|awk '{print $3 }'|xargs docker rmi
```
给个执行权限：
```dtd
chmod +x restartWeb.sh
```

# 后端
```dtd
我先用 spring 随便搭个小程序后台
```
## 编写 docker-compose
先创建一个 mysql 目录作为持久化目录：
```dtd
mkdir -p /root/mysql_home
```
我需要一个 mysql + 自己的 app 镜像：
```dtd
[root@VM-0-9-centos ~]# cat docker-compose.yml
version: '2.2'
services:
    mysql:
        image: mysql:5.7
        volumes:
            - "~/mysql_home/data:/var/lib/mysql"           # 挂载数据目录
            - "~/mysql_home/config:/etc/mysql/conf.d"      # 挂载配置文件目录
            - "~/mysql_home/mysql-init.sql:/tmp/mysql-init.sql"
        environment:
            - MYSQL_ROOT_PASSWORD=123qwe
            - MYSQL_USER=root
            - MYSQL_PASSWORD=123qwe
        ports:
            - "13307:3306"
        expose:
            - "3306"

    dalihaozai:
        image: docker.io/ouyangbro/dalihaozai:latest
        environment:
            SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/dalihaozai?autoReconnect=true&useSSL=false&useUnicode=true&characterEncoding=UTF-8
            SPRING_DATASOURCE_PASSWORD: 123qwe
            calendar.monthCnt: 3
            cms.login.password: 46f94c8de14fb36680850768ff1b7f2a
        ports:
            - "8080:8080"
        links:
            - mysql
```
每次拉完镜像后；
```dtd
docker pull docker.io/ouyangbro/dalihaozai:latest
```
第一次启动：
```dtd
docker-compose up -d
```
重启：
```dtd
[root@VM-0-9-centos ~]# docker-compose restart
Restarting root_dalihaozai_1 ... done
Restarting root_mysql_1      ... done
```
接下来在自己的 mac 上，写好程序，打镜像：
放在项目根目录下的 dockerFile（和 pom.xml 同级）:
```dtd
FROM frolvlad/alpine-oraclejdk8:slim
VOLUME /tmp
ADD ./target/dalihaozai-1.0.0-SNAPSHOT.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```
推送：
```dtd
打镜像：
docker build -t dalihaozai:latest .

打镜像tag：
docker tag dalihaozai:latest ouyangbro/dalihaozai:latest

推到docker服务器：
docker login
docker push ouyangbro/dalihaozai:latest
```