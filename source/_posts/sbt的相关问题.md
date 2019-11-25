---
title: sbt的相关问题
date: 2018-09-20 10:30:47
tags:
  - 大数据
description: sbt的相关问题
toc: true
copyright: true
---

## 一、sbt下载依赖太慢

sbt类似一个maven工具，需要配置环境变量

[sbt下载链接](https://piccolo.link/sbt-0.13.18.tgz)

```shell
cd /usr
#创建文件夹
sudo mkdir sbt
cd ~/下载
#移到到创建的/usr/sbt文件夹
sudo mv sbt-0.13.18.tgz /usr/sbt
#解压文件
sudo tar -zxvf sbt-0.13.18.tgz
#编辑当前用户环境变量配置
vim ~/.bashrc
#配置sbt环境配置
export SBT_HOME=/usr/sbt/sbt
export PATH=${SBT_HOME}/bin:$PATH
#保存配置
cd ~
source .bashrc
```

#### 1、可以本地设置全局代理proxychains

```shell
#下载并安装全局代理工具 
apt install proxychains
#
cd ~
#创建proxychains隐藏文件夹
mkdir .proxychains
#新建全局代理配置文件
touch proxychains.conf
```

```yaml
#编辑全局代理配置文件
# proxychains.conf  VER 3.1
#
#        HTTP, SOCKS4, SOCKS5 tunneling proxifier with DNS.
#	

# The option below identifies how the ProxyList is treated.
# only one option should be uncommented at time,
# otherwise the last appearing option will be accepted
#
#dynamic_chain
#
# Dynamic - Each connection will be done via chained proxies
# all proxies chained in the order as they appear in the list
# at least one proxy must be online to play in chain
# (dead proxies are skipped)
# otherwise EINTR is returned to the app
#
strict_chain
#
# Strict - Each connection will be done via chained proxies
# all proxies chained in the order as they appear in the list
# all proxies must be online to play in chain
# otherwise EINTR is returned to the app
#
#random_chain
#
# Random - Each connection will be done via random proxy
# (or proxy chain, see  chain_len) from the list.
# this option is good to test your IDS :)

# Make sense only if random_chain
#chain_len = 2

# Quiet mode (no output from library)
#quiet_mode

# Proxy DNS requests - no leak for DNS data
proxy_dns 

# Some timeouts in milliseconds
#tcp_read_time_out 15000
#tcp_connect_time_out 8000

# ProxyList format
#       type  host  port [user pass]
#       (values separated by 'tab' or 'blank')
#
#
#        Examples:
#
#            	socks5	192.168.67.78	1080	lamer	secret
#		http	192.168.89.3	8080	justu	hidden
#	 	socks4	192.168.1.49	1080
#	        http	192.168.39.93	8080	
#		
#
#       proxy types: http, socks4, socks5
#        ( auth types supported: "basic"-http  "user/pass"-socks )
#
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks4 	127.0.0.1 9050
```

#### 2、通过代理重新下载依赖

切换到spark-demo项目目录下，然后一个个module模块进行编译。下载依赖。这样导入idea会比较快一点。

```shell
cd ~/桌面
cd source/
cd spark-demo/
#通过全局依赖插件，下载sbt依赖
proxychains sbt log/assembly
proxychain sbt demo/assembly
proxychains sbt data/assembly
proxychains sbt test/assembly
```

