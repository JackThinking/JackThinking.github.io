---
title: 深入理解Java虚拟机-Chapter10
date: 2018-08-15 10:10:44
categories: 
    - JVM虚拟机
tags:
    - java
    - JVM
---
## 早期（编译期）优化概述

**JVM的编译器可以分为三个编译器**：

* 前端编译器：把.java转变为.class的过程。如Sun的Javac、Eclipse JDT中的增量式编译器（ECJ）。
* JIT编译器：把字节码转变为机器码的过程，如HotSpot VM的C1、C2编译器。
* AOT编译器：静态提前编译器，直接将*.java文件编译本地机器代码的过程。

**本节讲述的仅限于第一类编译过程**

> Java即时编译器在运行期的优化过程对于程序运行来说更加重要，而前端编译器在编译期的优化过程对于程序编码，来说关系更加密切。

## Javac编译器

Javac编译器本身是由Java语言编写的程序。

编译过程大致可以分为3个过程：

* 解析与填充符号表过程。
* 插入式注解处理器的注解处理过程。
* 分析与字节码生成过程。
  
这3个步骤之间的关系与交互顺序如下：
[![JVM之编译器优化1.jpg](https://i.loli.net/2018/08/15/5b739229dc4e3.jpg)](https://i.loli.net/2018/08/15/5b739229dc4e3.jpg)

### 解析与填充符号表

解析步骤包括了词法分析和语法分析两个过程

#### 词法分析与语法分析

**词法分析**：将源代码的字符流转变为标记（Token）集合，单个字符是程序编写过程的最小元素，而标记则是编译过程的最小元素，关键字、变量名、字面量、运算符都可以成为标记。如“int a=b+2”这句代码就包含了6个标记，分别是int、a、=、b、+、2。

**语法分析**：是根据Token序列构造抽象语法树的过程，「抽象语法树」是一种用来**描述程序代码语法结构的树形表述方式**。语法树的每一个节点都代表着程序代码中的一个语法结构，例如包、类型、修饰符、接口、返回值甚至代码注释都可以是一个语法结构。

**经过这个步骤之后，编译器就基本不会再对源码文件进行操作了，后续的操作都是建立在抽象语法树之上的。**

#### 填充符号表

完成抽象语法树之后，下一步就是填充符号表的过程，即enterTrees()方法。符号表是由一组符号地址和符号信息构成的表格，类似于哈希表中K-V值对的形式。符号表中所登记的信息在编译的不同阶段都要用到。当对符号名进行地址分配时，符号表是地址分配的依据。

### 注解处理器

JDK1.5之后，Java提供了对注解的支持，这些注解与普通的Java代码一样，在运行期间发挥作用。

可以把它看做是一组编译器的插件，在这些插件里面，可以读取。修改、添加抽象语法树中的任意元素。如果这些插件在处理注解期间对语法树进行了修改，编译器将回到解析及填充符号表的过程重新处理，直到所有的插入式注解处理器都没有再对语法树进行修改为止，每一次循环称为一个Round。也是上图中的回环过程。

### 语义分析与字节码生成

语法分析之后，编译器获得了程序代码的抽象语法树表示，语法树能表示一个结构正确的源代码抽象。而语义分析的主要任务是对结构上正确的源程序进行上下文有关性质的审查，如进行类型审查。
在Javac编译过程中，语法分析过程分为**标注检查**以及**数据及控制流分析**两个步骤。

* 标注检查：
  标注检查步骤检查的内容包括诸如：变量使用前是否已被声明、变量与赋值之间的数据类型是否能够匹配等。此外，这个过程中还有一个重要的步骤称为常量折叠，如定义
  int a = 1 + 2 和 int a = 1 + 2 是一样的
* 数据及控制流分析：
  数据及控制流分析是对程序上下文逻辑更进一步的验证，它可以查出诸如程序员局部变量在使用前是否有赋值、方法的每条路径是否都有返回值、是否所有的受查异常都被正确处理了等问题。
  编译期的数据及控制流分析与类加载时的数据及数据流分析的目的基本上是一致的，但校验范围有所区别，有一些校验项只有在编译期或者运行期才能进行。如将局部变量声明为final，对运行期是没有影响的，变量的不变性仅仅由编译器在编译期间保障。
* 解语法糖：
  语法糖是指在计算机语言中添加某种语法，这种语法对语言的功能并没有影响，但是更方便程序员使用。
  Java是一种“低糖语言”，常用的语法糖主要是之前提到的泛型、变长参数、自动装箱/拆箱等。虚拟机运行时不支持这些语法，它们在编译期还原回简单的基础语法结构，这个过程称为解语法糖。
* 字节码生成：
  字节码生成是Javac编译过程的最后一个阶段，字节码生成阶段不仅仅是把前面各个步骤所生成的信息（语法树、符号表）转化为字节码写入磁盘中，编译器还进行了少量代码添加和转换工作。
  实例构造器< init >方法和类构造器< clinit >方法就是在这个阶段添加到语法树中的。

## Java语法糖

### 泛型和类型擦除

与C#的泛型不一样的是，Java的泛型只存在于程序源码中，**在编译后的字节码文件中，就已经替换成原来的原生类型，也称为裸类型，并且在相应的地方插入了强制转型代码。**

对于运行期的Java语言来说，`ArrayList< String >`与`ArrayList< Integer >`就是同一个类，所以**泛型技术实际上是Java语言的一颗语法糖，Java语言中的泛型实现方法称为类型擦除，基于这种方法实现的泛型称为伪泛型。**

故当方法名一样，`List< String >`和List`< Integer >`作为参数时，擦除使得两者的特征签名变得一样，导致拥有这两个方法无法重载。

擦除法所谓的擦除，仅仅是对方的Code属性中的字节码进行擦除，实际上元数据中还是保留了泛型信息，这也是我们能通过反射手段取得参数化类型的根本依据。

### 自动装箱、拆箱与遍历循环

自动装箱、拆箱在编译之后就被转换成了相应的包装和还原方法，如Integer.valueOf()与Integer,intValue()方法，而遍历循环则把代码还原成了迭代器的实现，这也是为何遍历循环需要被遍历类实现Iterable接口的原因。
**栗子：**

```java
public class Main {
    public static void main(String[] args) {
        Integer a = 1;
        Integer b = 2;
        Integer c = 3;
        Integer d = 3;
        Integer e = 321;
        Integer f = 321;
        Long g = 3L;
        Long h = 2L;
         
        System.out.println(c==d);
        System.out.println(e==f);
        System.out.println(c==(a+b));
        System.out.println(c.equals(a+b));
        System.out.println(g==(a+b));
        System.out.println(g.equals(a+b));
        System.out.println(g.equals(a+h));
        System.out.println(a==b);
    }
}
```

输出结果：

```java
true
false
true
true
true
false
true
false
```

当包装器类进行“==”比较时，内部会调用 Integer.valueOf方法进行自动装箱（int -> Integer），源码如下：

```java
/**
   * Returns an {@code Integer} instance representing the specified
   * {@code int} value.  If a new {@code Integer} instance is not
   * required, this method should generally be used in preference to
   * the constructor {@link #Integer(int)}, as this method is likely
   * to yield significantly better space and time performance by
   * caching frequently requested values.
   *
   * This method will always cache values in the range -128 to 127,
   * inclusive, and may cache other values outside of this range.
   *
   * @param  i an {@code int} value.
   * @return an {@code Integer} instance representing {@code i}.
   * @since  1.5
   */
  public static Integer valueOf(int i) {
      if (i >= IntegerCache.low && i <= IntegerCache.high)
          return IntegerCache.cache[i + (-IntegerCache.low)];
      return new Integer(i);
  }
```

从源码中可见，Integer对象内部有IntegerCache类，可缓存（-128~127范围的数值），如果超过了，则会返回一个新的Integer类。由于“==”比较的是内存地址，因此，在“-128~127”数值范围内，比较的是同一个对象，得到true，而超过了该范围，则是返回自动装箱后的新对象，因此得到false。

**总结：**

* Integer（-128~127）、Short（-128~127）、Byte（-128~127）、Character（0~127）、Long（-128~127）这几个包装类的valueOf方法的实现是类似的
* Double、Float的valueOf方法的实现是类似的，并没有缓存，直接返回一个新的实例化对象
* Boolean的valueOf方法的实现是个三目运算，形如return (b ? TRUE : FALSE); TRUE、FALSE为两个内部定义的静态成员，这里直接返回两者其一

* 当 “==”运算符的两个操作数都是 包装器类型的引用，则是比较指向的是否是同一个对象，而如果其中有一个操作数是表达式（即包含算术运算或含有基本类型）则比较的是数值（即会触发自动拆箱的过程）。另外，对于包装器类型，equals方法并不会进行类型转换。
