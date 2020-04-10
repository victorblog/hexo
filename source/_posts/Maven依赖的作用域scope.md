---
title: Maven作用域scope分类
date: 2016.09.20 18:30:47
tags:
  - Java
description: Maven作用域scope分类
toc: true
copyright: true
---

### 1、Maven中依赖的作用域scope的分类

Maven默认的依赖配置项中，scope的默认值是compile

##### 1.1、compile

<font color="red">默认是compile</font>，也就是什么都不配置，就意味着compile，表示被依赖的项目需要参与当前项目的编译，后续的测试，运行周期也会使用到。是一个比较强的依赖，打包的时候也需要含进去。

##### 1.2、runtime

表示依赖项目无需参与项目的编译，不过后期的测试和运行周期需要参与。与compile相比，只是跳过编译而已。比较常见的是比较常见的如JSR-xxxx的实现，对应的API jar是compile的，具体实现是runtime的，compile只需要知道接口就足够了。oracle jdbc驱动架包就是一个很好的例子，一般scope为runntime。另外runntime的依赖通常和optional搭配使用，optional为true。我可以用A实现，也可以用B实现。

##### 1.3、provided

provided意味着打包的时候可以不用包进去，别的设施(Web Container)会提供。事实上该依赖理论上可以参与编译，测试，运行等周期。相当于compile，但是在打包阶段做了exclude的动作。

##### 1.4、test

该依赖仅仅参与测试相关的内容，包括测试用例的编译和执行，比如定性的Junit。