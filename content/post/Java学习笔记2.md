---
title: Java学习笔记2
date: 2018-04-25 15:42:06
categories: 
    - Java
tags:
    - java
---

这次将书中第三章的其余内容进行扫尾工作，主要是详细介绍数组为主。
## 数组
之前在做LeetCode的时候，就发现大量涉及到字符串和数组的操作，所以在此主要介绍一下Java中的数组操作。
在声明数组的之后需要初始化才可以使用，例如：
``` java
int[] a;
int[] a = new int[100];
```
第一句只有声明，第二句使用new之后才真正创造出了数组。
**注意**:一旦创建了数组，就不能改变它的大小，如果想改的话用数组列表（array list）。
### for each循环
Java里面有一种和python中（for word in words）类似的很好用的循环语句。Java中的用法是如下：
 ``` java
for each word in words
```
这种写法更加简洁且不易出错。
### 数组初始化以及匿名函数
 ``` java
int[] a = {1,2,3,4,5};
int[] a = new int[100];
```
第一种方法不需要new，直接赋值即可；第二种方法需要。
### 数组拷贝
 ``` java
int[] a = b;
a[5] = 12;
```
这样a,b两个变量引用同一个数组，相当于其指针指向的数据区是一样的。
如果希望拷贝到一个新的数组而不是公用一个数组的时候，可以使用array类的copyto方法，例如：
 ``` java
int[] a = Arrays.copyOf(a,a.length);
```
### 数组排序
Array类中有sort方法，如下：
 ``` java
int[] a = new int[100];
Arrays.sort(a);
```
方法使用了快速排序算法。
### 多维数组
没啥好说的，和其它语言类似，不过想要快速打印二维数组的数据元素列表的话，有个函数如下：
 ``` java
System.out.println(Arrays.deepToString(a));
```
## 控制流程&大数值
1.不能在嵌套层的两个块中声明同名的变量
2.case分支语句的末尾没有break语句，会执行下一个case分支语句，这种情况非常危险
3.Java额外提供了一种带标签的break语句，用于跳出多重嵌套的循环语句

4.与C++不同，Java没有提供运算符重载功能
