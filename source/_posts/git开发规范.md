---
title: git开发规范
date: 2017-11-03 15:23:33
tags:
- Git
toc: true
copyright: true
---

### 1、配置SSH

``` shell
#配置用户名，邮箱
git config --global user.name "xxx"
git config --global user.email "xxx@163.com"
#设置pull代码为rebase
git config --global --bool pull.rebase true
#设置git log的格式
alias gl='git log --graph --pretty=format:'\''%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset'\'' --abbrev-commit --date=relative'

#生成秘钥
ssh-keygen -t rsa 
#查看公钥
cat ~/.ssh/id_rsa.pub
#把key赋值到git的settings的SSH keys
```

### 2、rebase代码

​		除了master分支，其他分支拉最新master代码，要加上rebase，保证代码整洁性，都在一条线上。

```shell
#切换到当前你自己的分支，获取最新的代码
git branch consult
#rebase远程master分支的最新代码，合并到你自己的分支
git rebase origin/master
#如果出现冲突，则需要解决冲突
git diff -w
#解决冲突文件
vim 冲突文件
git rebase origin/master
git rebase --continue
git add .
git log
#然后fetch到当前分支
git fetch
#然后push到自己的分支
git push origin consult
#push不上去的时候，直接强push(慎用)
git push -f origin consult

```

### 3、合并自己的分支代码到master

​		合并自己分支的代码到master的时候，需要git merge --no-ff xxx

``` shell
#切换分支到master
git checkout master
#合并自己分支代码到master分支
git merge --no-ff consult
#合并完之后，查看下日志，有没有自己分支的代码
git log
#查看分支状态
git status
#合并都增加准备提交
git add .
#本地提交
git commit -m "合并分支consult"
#然后远程push到master分支
git push origin master

```



 

