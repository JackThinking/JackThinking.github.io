---
title: Java学习笔记5
date: 2018-05-18 20:20:02
categories: 
    - Java
tags:
    - java
---
还是要补充基础知识啊

## 反射

## 泛型
### 简单泛型类
```java
public class Pair<T> 
{
   private T first;
   private T second;

   public Pair() { first = null; second = null; }
   public Pair(T first, T second) { this.first = first;  this.second = second; }

   public T getFirst() { return first; }
   public T getSecond() { return second; }

   public void setFirst(T newValue) { first = newValue; }
   public void setSecond(T newValue) { second = newValue; }
}
```
这样的话
```java
Pair<String>()
Pair<String>(String,String)
```
泛型类可以看做是普通类的工厂

### 泛型方法
```java
class Array
{
    public static <T> T getmid(T...a)
    {
        return null;
    }
}
```
1.泛型方法可以定义在普通类中，也可以在泛型类中
### 泛型变量的限定
```java
public static <T extends Comparable> T getmid(T...a)
    {
        return null;
    }
```
### 泛型代码和虚拟机

### 约束和局限性

### 泛型类型的继承规则

## 多线程

## 枚举

## 内部类
### 原因
1.内部类方法可以访问该类定义所在的作用域中的数据，包括私有数据
2.内部类可以对同一个包中的其他类预隐藏起来
3.定义一个回调函数且不想编写大量代码时，使用匿名内部类比较便捷

### 特性
1.内部类既可以访问自己的数据域，也可以访问外围类对象的数据域
2.outerclass.this代表正规的外部引用
3.outerobject.new innerclass(construction parameters)
4.局部内部类：可以在方法中定义内部类（不能用public或者private进行声明，对外部世界完全隐藏）
