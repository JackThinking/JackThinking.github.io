---
title: Java学习笔记3
date: 2018-04-30 09:08:55
tags:
    - java
---
之前在慕课网上学完《Spring Boot进阶之Web进阶》和《2小时学会Spring Boot》中间遇到了不少java语法上的问题，不过当时就先跳过了，现在把之前不会的知识点进补充学习。

## 对象与类
Java在面向对象上比C++做的更加彻底，下面来介绍一下。
### 类之间的关系
1.依赖（uses-a）: 一个类的方法操作另外一个类的对象
2.聚合（has-a）:类A的对象包含类B的对象
3.继承（is-a）:表示特殊与一般关系的

### 对象与对象变量
1.一个对象变量并没有实际包含一个对象，而仅仅是引用一个对象（C++中指针生成对象的感觉）
``` java
Date birthday;//java
Date* birthday;//C++
```
2.指针没有初始化的话，会直接报错，而不是产生随机数，不必担心内存管理问题
3.必须使用clone方法获得对象的完整拷贝

### 用户自定义类

1.一个源文件，只有有一个公有类，但是可以有任意的私有类
2.建议类内变量设为private，方法（一般）设为public

### 构造函数
1.构造器没有返回值
2.构造器总是伴随着new操作仪器调用
3.在方法中不要命名与实例域（类中前面定义的）同名的变量

### 隐式参数与显式参数
在每一个方法中，关键字this表示隐式参数
``` java
public void abc(double d){
    double e = this.g * d;
    this.g += e;
}
```
有些程序员喜欢这个风格，因为可以讲局部变量和实例域明显的区分开来。
Java程序的方法定义都是在类内部，但这表示他们是内联方法。

### 静态域和静态方法
1.实例域定义为final，构建对象时必须初始化这样的域
2.域定义为static的话，每个类只有只有一个这样的域，而每个对象的所有实例域都有自己的拷贝
3.静态变量使用的少，静态常量使用的多，比如π
4.静态方法是一种不能对对象进行操作的方法，即没有this参数
5.静态方法可以访问吱自身类中的静态域
6.可以使用对象来调用静态方法，但是建议使用类名，而不是对象

### 方法参数
1.Java程序设计对对象引用是值传递
2.一个方法不能修修改一个基本数据类型的参数
3.一个方法可以改变一个对象参数的状态
4.一个方法不能让对象参数引用一个新的对象

### 类设计技巧
1.一定要保证数据私有
2.一定要对数据初始化.
3.不要在类中使用过多的基本类型
4.不是所有的域都需要独立的域访问器和域更改器
5.将职责过多的类进行分解
6.类名和方法名要能体现他们的职责

## 接口

Java中，接口不是类，而是对类的一组需求描述。
### 接口的特性
比如下面的Flyable接口就定义了三个方法：
``` java
public interface Runner {void run();}

public interface Swimmer {void swim();}
public interface Flyable { 
	abstract void fly(); 
	void land(); 
	void takeoff();
}
```
1.接口的所有方法都自动属于public
2.接口不能实例化，而且也不能在接口内实现方法
3.实现接口的时候一定要声明为public
为了让类实现接口，通常需要以下两个步骤：
1.对类声明为实现给定的接口
2.对接口中的所有方法进行定义
3.用关键词implements
``` java
abstract class Animal  {
	int age;
	abstract public void eat();
}

class Person extends Animal implements Runner,Swimmer,Flyable {
	public void run() { System.out.println("run"); }
	public void swim()  { System.out.println("swim"); }
	public void eat() { System.out.println("eat"); }
	public void fly() { System.out.println("fly");}
	public void land(){ System.out.println("land");}
	public void takeoff(){ System.out.println("takeoff");}
}
```
### 接口与抽象类
接口与抽象类有点相似，为什么不直接用抽象类？
比如：
``` java
abstract class Comparable{
    public abstract int compareTo(Object)
}
class Employee extends Comparable{
    public int compareTo(Object other){...}
}
```
因为抽象类有个问题，每个类只能拓展与一个类，下面的例子：
``` java
class Employee extends Comparable，Person //error

class Employee extends Comparable implements Person //ok
}
```
实际上，接口可以提供多重继承的大多数好处，同时还能避免多重继承的复杂性和低效性。

## 泛型

泛型总体来说有点像C++的模板，作用是对于不同的参数类型实现相同的方法，但是本质上Java的泛型和C++的模板还是很不一样的。

### 定义简单的泛型类
```java
import java.util.*;

class GenericTreeClass {
	public static void main(String[] args){ 
		TNode<String> t = new TNode<>("Roo");
		t.add("Left"); t.add("Middle"); t.add("Right");
		t.getChild(0).add("aaa");
		t.getChild(0).add("bbb");
		t.traverse();
	}
}

class TNode<T>
{
  private T value;
  private ArrayList<TNode<T>> children = new ArrayList<>();

  TNode(T v) { this.value = v; } 
  public T getValue() { return this.value; } 
  public void add(T v) {
    TNode<T> child = new TNode<>(v);
    this.children.add(child);
  }
  public TNode<T> getChild(int i) {
    if ((i < 0) || (i > this.children.size())) return null;
    return (TNode<T>)this.children.get(i);
  }

  public void traverse() {
    System.out.println(this.value);
    for (TNode child : this.children)
      child.traverse();
  }
}
```

### 泛型方法
泛型方法既可以定义在普通类中，也可以定义在泛型类中，<T>要放在函数的前面。
```java
import java.util.*;

class GenericMethod {
	public static void main(String[] args){ 
		Date date = BeanUtil.<Date>getInstance("java.util.Date");
		System.out.println(date);
	}
}

class BeanUtil{
	public static <T> T getInstance( String clzName ){
		try
		{
			Class c = Class.forName(clzName);
			return (T) c.newInstance();
		}
		catch (ClassNotFoundException ex){}
		catch (InstantiationException ex){}
		catch (IllegalAccessException ex){}
		return null;
	}
}
```

### 类型变量的限定
1.使用？
```java
reverse(List<?> list)
```
2.使用extends
```java
addAll(Collection<? extends E> col)
```
3.使用super
```java
fill(List<? super T>list,T obj)
```
总体来说，泛型是比较高级的机制，有点难，之后有需求再详细学习。
