---
title: JVM相关
date: 2019-11-20 10:46:34
tags:
- Java
toc: true
copyright: true
---

## 一、JVM相关的笔记

简历上可以写，熟悉JVM架构与GC垃圾回收机制，以及相应的参数调优，有过在Linux上进行系统优化的经验。

面试官让你谈谈jvm优化，我是根据淘宝周志明那哥们写的那本深入理解java虚拟机书，99*%的优化的是堆Heap，1%的优化的是方法区Method Area，虚拟机栈Stack是私有的，不可能优化，最多是栈溢出StackOverFlow，调整一下栈的大小<font color=red>-xss参数</font>
<font color=red>静态变量+常量+类信息+运行时常量池</font>存在方法区。方法区一般放的是<font color=red>类的构造函数和接口的代码。所有定义的方法的信息都保存在方法区。</font>
引用存在虚拟机栈里。实例变量存在堆里面。

### 1、栈

​    栈，也叫栈内存，主管java程序的运行，是在线程创建是创建，它的生命周期跟随线程的生命周期，线程结束栈内存也就释放，对于栈来说，不存在垃圾回收，生命周期和线程一致，是线程私有的。
   基本类型变量和对象的引用变量，都是在函数的栈内存里分配。

    #### 1.1、栈存储的是什么？
   栈是先进后出，比如你main方法调用test1，test1调用test2，等等，所以main在最底下，可以看作一个栈帧对应一个方法。
    栈帧里面存3种类型变量：

- 本地变量（local variables），输入参数和输出参数，以及方法内的变量。
- 栈操作（ operand stack）出栈和入栈的操作
- 栈帧数据（Frame data）：包括类文件，方法等，言下之意，每一个方法就是一个栈帧。

``` java
public class{
    //test的栈帧在上面，直接如此调用，把栈给撑爆了，所以栈溢出
    public void test(){
          test();
     }
    //main方法的栈帧在最底部
    public static void main(String[] Args){
         test();
     }
}
 //死循环，或者循环递归调用,出现的异常java.lang.StackOverflowError
```

### 2、堆Heap

​    一个JVM实例只存在一个堆内存，堆内存大小是可以调节的。类加载器读取了类文件后，需要把类、方法、成员变量存在堆内存中，保存所有引用类型的真实信息，以方便执行器执行，堆内存分为三个部分：
   ####  2.1、Young generation space 新生代 young

新生代分为3个区：

- Eden space伊甸区
- Survivor 0 space 幸存0区
- Survivor 1 space 幸存1区

#### 2.2、Tenure generation space 养老区 Old

#### 2.3、Permanent space 永久区 Perm
## 二、3种JVM

- SUN公司的HotSpot虚拟机
- BEA公司的JRockit虚拟机
- IBM公司的J9VM虚拟机