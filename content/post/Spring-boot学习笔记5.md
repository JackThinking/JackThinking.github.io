---
title: Spring boot学习笔记5
date: 2018-05-06 09:51:30
categories: 
    - Spring boot
tags:
    - Spring boot
---
记录在spring boot学习中遇到的知识和难点

## 小知识
1.lombok的Data,Getter,Setter的区别就是，Data包含（Getter&Setter）
2.在使用BigDecimal的时候他的计算方法都是有函数的（multiply（）而不是*）
3.数据库对象DAO在实际的与前端通信的过程中不能直接拿来用，因为往往会设计数据库中没有的字段，比如我这个项目中，使用的DTO比DAO多了
```java
private List<OrderDetail> orderDetailList ;
```
这个字段需要在后端处理的时候用到，但又不是DAO的基本字段
4.随机数生成方法（六位）：
```java
public static synchronized String getUniqueKey(){
    Random random = new Random();
    Integer number = random.nextInt(900000)+100000;//生成六位随机数

    return System.currentTimeMillis()+String.valueOf(number);//int转string
}
```
5.synchronized字段是为了防止多线程的时候随机数重复

## 小技巧
1.cmd+shift+u:切换大小写

## 小问题
1.BeanUtil的使用不会
2.Lambda表达式的使用不会
