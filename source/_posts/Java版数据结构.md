---
title: Java数据结构
date: 2017.07.02 10:06
tags:
  - Java
description: Java数据结构
toc: true
copyright: true
---

# 1、数据结构的特性

| 数据结构 | 优点                                                       | 缺点                                                     |
| -------- | ---------------------------------------------------------- | -------------------------------------------------------- |
| 数组     | 插入快，如果知道下标，可以非常快速的存取                   | 删除慢，大小固定                                         |
| 有序数组 | 比无序的数组查找快                                         | 删除和插入慢，大小固定                                   |
| 栈       | 先进后出的存取方式                                         | 存取其他项很慢                                           |
| 队列     | 先进先出的存取方式                                         | 存取其他项很慢                                           |
| 二叉树   | 查找、插入、删除都快（如果树保持平衡）                     | 删除算法复杂                                             |
| 红-黑树  | 查找、插入、删除都快。树总是平衡的                         | 算法复杂                                                 |
| 2-3-4树  | 查找、插入、删除都快、树总是平衡的。类似的树对磁盘存储有用 | 算法复杂                                                 |
| 哈希表   | 如果关键字已知，则存储极快，插入快                         | 删除满，如果不知道关键字则存储很慢，对存储空间使用不充分 |
| 堆       | 插入、删除快。对大数据项的存取很快                         | 对其他数据项存取慢                                       |
| 图       | 对现实世界建模                                             | 有些算法慢且复杂                                         |

## 1.1、数组

- 创建数组

  ```java
  /*
  Java中数组作为对象处理。创建数组使用new关键字
  一旦创建数组，数组大小便不可改变
  */
  int[] arr=new int[10];
  ```

- 访问数组数据项

  ```java
  /*数组数据项通过方括号的下标来访问*/
  arr[0]=100;
  ```

- 数组初始化

  ```java
  int[] arr={1,2,3,4,5};
  ```

## 1.2、有序数组

- 有序数组特点

  有序数组按照数组元素一定的顺序排列，方便使用二分查找来查找数组中特定的元素。有序数组提高了查询效率，并没有提高删除和插入元素的效率。

## 1.3、查找算法

- 线性查找

  查找的时候，将要查找的数一个个的与数组中的元素比较，直到找到要找的数。

- 二分查找（折半查找）

  不断的将有序数组进行对半分割，每次拿中间位置的数和要查找的数进行比较：

  - target<middle,target在数组的前半段。
  - target>middle,target在数组的后半段
  - target==middle,target就是中间数

  ```java
  /*
  二分查找索引（前提：有序）
  */
  public int binarySearch(long value){
  	int middle=0;
  	int left=0;
  	int right=size-1;
      while(true){
          middle=(left+right)/2;
          if(arr[middle]==value){
              return middle;
          }else if(left>right){
              return -1;//不存在
          }else{
              if(arr[middle]>value){
                  right=middle-1;//在左半部分
              }else{
                  left=middle+1;//在右半部分
              }
          }
      }
  }
  ```

## 1.4、栈和队列

​	栈和队列都是抽象数据类型（Abstract Data Type，ADT），既可以用数组实现，又可以用链表实现。

### 1.4.1、栈

```java
public class TestStack{
    private long[] arr;
    private int top;
    public TestStack(){
        arr=new long[100];
        top=-1;
    }
    public TestStack(int maxSize){
        arr=new long[maxSize];
        top=-1;
    }
    public void push(long value){
        arr[++top]=value;
    }
    public long pop(){
        return arr[top--];
    }
    public long peek(){
        return arr[top];
    }
    public boolean isEmpty(){
        return (top==-1);
    }
    public boolean isFull(){
        return (top==arr.length-1);
    }
}
```

### 1.4.2、链表

- 链结点：一个链结点（Link，结点）是某个类的对象，该对象中包含对下一个链结点引用的字段（next）

  ```java
  /*结点定义*/
  public class Link{
      public long data;//data域
      public Link next;//next Link in list
      public Link(long data){
          this.data=data;
      }
      public void displayLink(){
          System.out.println(data+"--->");
      }
  }
  ```

- 单链表（LinkList）

  ```java
  public class LinkList{
      private Link first;//定义头结点
      public LinkList(){
          this.first=null;
      }
      public void insertFirst(long value){
          Link newLink=new Link(value);
          newLink.next=first;
          first=newLink;
      }
      public Link deleteFirst(){
          if(first==null){
              return null;
          }
          Link temp=first;
          first=first.next;
          return temp;
      }
      public void displayList(){
          Link current=first;
          while(current!=null){
              current.displayLink();
              current=current.next
          }
          System.out.println();
      }
  }
  ```

  

