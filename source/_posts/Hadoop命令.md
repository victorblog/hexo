---
title: Hadoop命令
date: 2017.09.13 10:06
tags:
  - 大数据
description: Hadoop命令
toc: true
copyright: true
---

### 1、文件操作

```shell
#在HDFS上创建目录
hadoop fs -mkdir <hdfs_path>
#查看文件
hadoop fs -ls <hdfs_path>
#显示文件的大小
hadoop fs -du <hdfs_path>
#只删除文件或者空目录
hadoop fs -rm <hdsf_path>
#递归删除目录，命令rm的递归版本，会递归删除目录下的各级目录以及文件
hadoop fs -rmr <hdfs_path>
```

### 2、查看文件内容

```shell
#显示文件内容
hadoop fs -cat <hdfs_path>
#将文件尾部1KB字节的内容输出到stdout
hadoop fs -tail [-f] URI
```

### 3、文件复制

```shell
#将文件源路径移动到目标路径
hadoop fs -mv <src1> <src2> <dest>
#将文件从源路径复制到目标路径
hadoop fs -cp <src1> <src2> <dest>       
#将本地文件放置在HDFS上
hadoop fs -copyFromLocal <local_path> <remote_path>     
#将HDFS上的文件复制到本地文件系统中
hadoop fs -copyToLocal <remote_path> <local_path>       
#复制文件到本地文件系统
hadoop fs -get <src> <localdst>    
#从本地文件系统中复制单个或多个源路径到目标文件系统中
hadoop fs -put <localsrc>... <dst>   
```

### 4、高权限

```shell
scp victor@192.168.10.1:/home/victor/py/test.py /usr/opt/script/
#去前1000行记录到新文件
head -1000 basic.dat > test1.dat            
```

