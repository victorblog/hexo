---
title: Linux常用命令
date: 2015.11.01 10:06
tags:
  - Linux
description: Linux常用命令
toc: true
copyright: true
---

#### Linux常用的操作命令

记住：Linux系统里一切都是文件

- 更新系统

  ```shell
  sudo apt update
  sudo apt upgrade
  ```

- 查看历史命令

  ```shell
  #清除历史
  history -c
  ```

- 查看某个命令的所在位置

  ```shell
  which ps
  ```

- 查看某个命令的使用文档

  ```shell
  man ps
  ```

- 查看端口占用情况：

  ```shell
  lsof -i:8080
  ```

- 查看某个进程：

  ```shell
  ps aux | grep "chrome"
  ```

- 杀死某个进程：

  ```shell
  kill -9 pid
  ```

- 展示MD5：

  ```shell
  echo "danruiqing" | md5sum
  ```

- 查看系统时间：

  ```shell
  date +%s
  date -d"1 week ago" +"%F %H:%M:%S"
  ```

- 查看路由表

  ```shell
  netstat -rn
  ```

- 路由表添加路由

  ```shell
  #添加路由
  sudo route add 10.249.7.38/32 dev wlp2s0
  ```

- 访问远程服务器

  ```shell
  ssh danruiqing@xxx.com -p 21
  ```

- 查看本地网络配置

  ```shell
  ifconfig -a
  ```

- maven清楚并忽略test

  ```shell
  mvn clean package -Dmaven.test.skip=true
  ```

- 查询命令的使用

  ```shell
  #grep "123"查询的结果作为后面的输入，管道
  grep "123" | grep "45" | grep "tom"
  ```

- 环境变量配置

  ```shell
  vim .bashrc
  export JAVA_HOME=/usr/java/jdk1.8
  export JRE_HOME=${JAVA_HOME}/jre
  export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/Lib
  export PATH=.${JAVA_HOME}/bin:$PATH
  :wq
  source .bashrc
  ```

- tar包的解压打包

  ```shell
  tar -zxvf xxx.tar
  tar -zcvf java_demo.tar java_demo
  ```

- zip包的解压打包

  ```shell
  zip java_demo.zip java_demo
  unzip -l example.zip
  ```

- 查看日志文件

  ```shell
  tail -f java.log
  cat .bashrc
  ```

  

