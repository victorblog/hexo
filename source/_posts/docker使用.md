---
title: Docker的使用
date: 2017.11.01 10:06
tags:
  - DevOps
description: Docker的使用
toc: true
copyright: true
---

# 一、docker的使用

#### 1、docker添加镜像加速文件

```shell
cd /etc/docker
#如果没有daemon.json文件，需要新建一个
sudo touch daemon.json
sudo vim daemon.json
#daemon.json添加以下配置，然后保存
{"registry-mirrors":["https://registry.docker-cn.com"]}
#重启服务
sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### 2、查看docker的版本

```shell
docker version
#使用docker查找镜像
sudo docker search mysql
#查看本地下载的镜像
sudo docker images
#查看本地正在运行的镜像
sudo docker ps -a
```

#### 3、使用docker拉取镜像

```shell
sudo docker pull redis:5.0.6
sudo docker pull nginx
sudo docker pull mysql:8.0.18
#删除拉取的镜像
sudo docker image rm mysql:8.0.18
```

#### 4、使用docker安装并运行nginx

```shell
sudo docker search nginx
#pull的时候，后面不追加冒号和版本号，默认是最新的latest版本
sudo docker pull nginx
#运行nginx
sudo docker run --name victor-nginx-test -p 8081:80 -d nginx
#victor-nginx-test是容器名称
-d，设置容器在后台一直运行
-p，端口进行映射，将本地的8081端口映射到容器内部的80端口
#执行玩上面那条命令后，会生成一串字符串45faac1c8dcac4c356e4fd8f8dc0ee204385feca7e851cbe874805db7b811f89，这个表示容器的id，一般作为日志的文件名来查看启动信息等
sudo docker logs 45faac1c8dcac4c356e4fd8f8dc0ee204385feca7e851cbe874805db7b811f89
#停止nginx
sudo docker ps
#复制对应容器的id,然后停止镜像
sudo docker stop 45faac1c8dca
#删除镜像
sudo docker rm 45faac1c8dca
```

#### 5、使用docker部署项目

```shell
#首先创建一个文件夹，里面包含3个子文件夹
sudo mkdir -p ~/桌面/nginx/www ~/桌面/nginx/logs ~/桌面/nginx/conf
#首先你要直到容器的id，
sudo docker ps -a
#复制docker容器里的nginx默认配置文件信息，到你的本地刚创建的conf文件夹目录下
sudo docker cp76d9d74d071e:/etc/nginx/nginx.conf ~/桌面/nginx/conf
#部署项目，到docker里的nginx
sudo docker run -p 8082:80 --name victor-nginx-web -v ~/桌面/nginx/www:/usr/share/nginx/html -v ~/桌面/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v ~/桌面/nginx/logs:/var/log/nginx -d nginx

#命令说明
-p 8082:80  是把你容器里的nginx的80端口映射到你本地主机的8082端口
--name victor-nginx-web  是把容器重命名我们自己喜欢的名字
-v ~/桌面/nginx/www:/usr/share/nginx/html   是把我们本地创建的项目 的www目录挂在到容器的/usr/share/nginx/html
-v ~/桌面/nginx/conf/nginx.conf:/etc/nginx/nginx.conf 是吧我们创建的nginx.conf挂在到容器的/etc/nginx/nginx.conf
-v ~/桌面/nginx/logs:/var/log/nginx 是把我们自己创建的logs挂在到容器的/var/log/nginx
-d nginx :是设置容器一直后台运行

#执行完命令后，会生成一串字符串，用这个字符串表示日志id，可以使用
sudo docker logs 76d9d74d071e8b2d8499a56691587eef3e2f19d4fddeb5a3e26e5d9338856e10
```

#### 6、使用docker安装mysql并运行

```shell
#拉去mysql的镜像
sudo docker pull mysql:5.7
#本地创建一个文件夹，用来映射容器的mysql
sudo mkdir -p ~/桌面/mydata/mysql/log ~/桌面/mydata/mysql/conf ~/桌面/mydata/mysql/data
#给创建好的文件夹开权限
sudo chmod 777 -R ~/桌面/mydata/
#使用docker命令启动mysql
sudo docker run -p 3305:3306 --name mysql -v ~/桌面/mydata/mysql/log:/var/log/mysql -v ~/桌面/mydata/mysql/data:/var/lib/mysql -v ~/桌面/mydata/mysql/conf:/etc/mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
#查看镜像，如果存在，或者端口被占用，则换为3305映射容器的端口
sudo docker ps -a
#先删除被占用名字的容器镜像id，然后重新执行启动命令
sudo docker rm3bf96e1f8a95
#进入运行mysql的docker容器
sudo docker exec -it mysql /bin/bash

########以下是mysql的命令操作
#使用mysql 命令打开客户端
mysql -uroot -proot --default-character-set=utf8
#查看数据库
show databases;
create database test character set utf8;
use test;
create table ();
#退出mysql命令操作
\q
#退出mysql运行的docker容器
exit
```

#### 7、使用docker安装并运行redis

```shell
#docker拉取redis
sudo docker pull redis:3.2
#本地创建一个redis文件夹存放数据
sudo mkdir -p ~/桌面/mydata/redis/data
#更改文件件执行权限
sudo chmod 777 -R ~/桌面/mydata/redis
#使用docker命令启动redis
sudo docker run -p 6379:6379 --name redis -v ~/桌面/mydata/redis/data:/data -d redis:3.2 redis-server --appendonly yes
#进入redis容器使用redis-cli命令进行连接
sudo docker exec -it redis redis-cli
#####以下为redis的命令
ping
set a 100
get a
#退出redis
exit
```

#### 8、使用docker安装并运行rabbitmq

```shell
#拉取docker镜像
sudo docker pull rabbitmq:3.7.15
#使用docker命令启动rabbitmq
sudo docker run --name rabbitmq --publish 5671:5671 --publish 5672:5672 --publish 4369:4369 --publish 25672:25672 --publish 15672:15672 --publish 15671:15671 -d rabbitmq:3.7.15
#Ubuntu的防火墙暴露端口15672
sudo ufw allow 15672
#开启防火墙
sudo ufw enable
#重新加载
sudo ufw reload
#查看防火墙状态
sudo ufw status
#进入到rabbitmq的连接
sudo docker exec -it rabbitmq /bin/bash
#然后输入如下命令，开启rabbitmq的管理后台
rabbitmq-plugins enable rabbitmq_management
#浏览器输入rabbitmq的管理后台页面
http://localhost:15672
用户名密码：guest guest
#退出
exit
#rabbitmq的下一次启动
#直接找到对应的Container ID
sudo docker ps -a
#然后启动命令
sudo docker start 213a1c20682e
#停止命令
sudo docker stop 213a1c20682e
```



