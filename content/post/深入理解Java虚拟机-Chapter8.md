---
title: 深入理解Java虚拟机-Chapter8
date: 2018-08-02 11:13:24
categories: 
    - JVM虚拟机
tags:
    - java
    - JVM
---
## 虚拟机字节码执行引擎

> 代码编译的结果是从本地机器码变成字节码，是存储格式发展的一小步，却是编程语言发展的一大步。

## 运行时栈帧结构

* 栈帧(Stack Frame)是用于支持虚拟机进行方法调用和方法执行的数据结构，它是虚拟机运行时数据区中的虚拟机栈(Virtual Machine Stack)的栈元素。
* 每一个栈帧都包含了**局部变量表**、**操作数栈**、**动态连接**、**方法返回地址**等信息。在编译程序代码的时候，栈帧中需要多大的局部变量表，多深的操作数栈都已经完全确定，并且写入到方法表的Code属性之中。
* 每一个**方法**从调用开始至执行完成的过程，都对应着一个**栈帧**在虚拟机栈里面从入栈到出栈的过程。
* 在活动线程中，只有位于栈顶的栈帧才是有效的。

### 局部变量表

* 局部变量表(Local Variable Table)是一组变量值存储空间，用于存放**方法参数**和方法内部定义的**局部变量**。
* 局部变量与类变量不同，**必须要赋初始值** 。

### 操作数栈

* 操作数栈(Operand Stack)也常称操作栈，它是一个后入先出(Last In First Out,LIFO)栈。操作数栈的最大深度也在编译的时候写如了Code属性的max_stacks数据项中。
* 当一个方法刚刚开始执行时，这个方法的操作数栈是空的，在方法的执行过程中，会有各种字节码指令往操作数栈中写入和提取内容，也就是出栈/入栈操作。
* 在概念模型中，两个栈帧作为虚拟机栈的元素，是完全独立的，但在大多数虚拟机的实现里都会做一些优化梳理，令两个栈帧出现一部分重叠。

### 动态连接

* 每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用，持有这个引用是为了支持方法调用过程中的动态连接(Dynamic Linking)。
* Class文件的常量池中存有大量的符号引用，字节码中的方法调用指令就以 **常量池中指向方法的符号引用作为参数** 。
* 这些符号引用一部分会在类加载阶段或第一次使用的时候就转化为直接引用，这种转化称为 **静态解析** 。另外一部分将在每一次运行期间转化为直接引用，这部分称为 **动态连接** 。

### 方法返回地址

* 当一个方法开始执行后，只有两种方式可以退出这个方法：
    * 第一种方式是执行引擎遇到任意一个 **方法返回的字节码指令** ，这时候可能会有返回值传递给上层的方法调用者（调用当前方法的方法称为调用者），是否有返回值和返回值的类型将根据遇到何种方法返回指令来决定，这种退出方法的方式称为正常完成出口(Normal Method Invocation Comletion)。
    * 另一种是，在方法执行过程中遇到异常，并且这个异常没有在方法体内得到处理，就会导致方法退出，这种退出方法的方式称为 **异常完成出口**(Abrupt Method Invocation Completion)。一个方法使用异常完成出口的方式退出，是不会给它的上层调用者产生任何返回值的。
* 一般来说，方法正常退出时，调用者的 **PC计数器的值** 可以作为返回地址，栈帧中很可能会保存这个计数器值。而方法异常退出时，返回地址是要通过 **异常处理器表** 来确定的，栈帧中一般不会保存这部分信息。
* 方法退出的过程实际上就等同于把当前栈帧出栈，因此退出时可能执行的操作有： 
    * 恢复上层方法的局部变量表和操作栈帧
    * 返回值（如果有的话）压入调用者栈帧的操作数栈中
    * 调整PC计数器的值 
    * 指向方法调用指令后面的一条指令等

### 附加信息

## 方法调用

**方法调用并不等同于方法执行**，方法调用阶段唯一的任务就是 **确定被调用方法的版本**（即调用哪一个方法） ，暂时还不涉及方法内部的具体运行过程。

### 解析

* 在Java语言中符合“编译期可知，运行期不可变”这个要求的方法，主要包括 **静态方法** 和 **私有方法** 两个大类，前者与类型直接关联，后者在外部不可被访问，这决定了它们不可能通过继承或别的方式重写其他版本，因此它们都适合 **在类加载阶段进行解析** 。
* 与之相对应的是，在Java虚拟机里面提供了5条方法调用字节码指令：
    * invokestatic：调用静态方法。
    * invokespecial：调用实例构造器方法、私有方法和父类方法。
    * invokevirual：调用所有的虚方法。
    * invokeinterface：调用接口方法，会在运行时再确定一个实现此接口的对象。
    * invokedynamic：先在运行时动态解析出调用点限定符所引用的方法，然后再执行该方法，在此之前的4条调用指令，分派逻辑是固化在Java虚拟机内部的，而invokedynamic指令的分派逻辑是由用户所设定的引导方法决定的。
* 只要能被`invokestatic`和`invokespecial`指令调用的方法，都可以在解析阶段中确定唯一的调用版本，符合这个条件的有 **静态方法** 、 **私有方法** 、 **实例构造器** 、 **父类方法** 4类，它们在类加载的时候就会把符号引用解析为该方法的直接引用，这些方法称为「非虚方法」。
* 还有一种特殊情况就是被final修饰的方法，虽然final方法使用invokevirual指令来进行调用，但是由于其无法被覆盖，没有其他版本，所以无须对方接受者进行多态选择。

### 分派！！！

Java具备面向对象的三个基本特征：继承，封装和多态

#### 静态分派——重载(Overload)的体现

* 重载：两同一不同
    * 类、方法名 相同
    * 形参列表 不同
    * 返回类型、修饰符 无关

```java
//方法静态分派演示
public class StaticDispatch{
  static abstract class Human{
  }
  static class Man extends Human{
  }
  static class Woman extends Human{
  }
  public void sayHello(Human guy){
    System.out.println("hello,guy!");
  }
  public void sayHello(Man guy){
    System.out.println("hello,getleman!");
  }
  public void sayHello(Woman guy){
    System.out.println("hello,lady!");
  }
  public static void main(String[] args){
    Human man = new Man();
    Human woman = new Woman();
    StaticDispatch sr = new StaticDispatch();
    sr.sayHello(man);   //输出hello,guy!
    sr.sayHello(woman);  //输出hello,guy!
  }
}
```

* 对于 `Human man = new Man()`; ：我们把 `Human` 称为变量的 **静态类型** (Static Type)，或者叫做变量的 **外观类型** (Apparent Type)，后面的 `Man` 则称为变量的 **实际类型** (Actual Type)。
* 静态类型的变化仅仅在使用时发生，变量本身的静态类型不会被改变，并且最终的静态类型是在编译期 **可知** 的；而 **实际类型** 变化的结果在 **运行期** 才可确定，编译器在编译程序的时候并 不知道 一个对象的实际类型是什么。
* 回到上述代码，main()里面的两次sayHello()方法调用，在方法接收者已经确定是对象“sr”情况下，使用哪个重载版本，就完全 **取决于传入参数的数量和数据类型** 。虚拟机（准确地说是编译器）在重载时是通过参数的 **静态类型** 而不是实际类型作为判定依据的。
* 所有依赖静态类型来定位方法执行版本的分派动作称为 **静态分派** 。

```java
//重载方法匹配优先级
public class Overload{
  public static void sayHello(Object args){
    System.out.println("hello object");  //优先级六，char装箱后转型为父类了，如果有多个父类，则在继承关系中由下往上搜索，越接近上层的优先级越低。
  }
  public static void sayHello(int args){
    System.out.println("hello int"); //优先级二，'a'可以代表数据97（'a'的Unicode数值为十进制数字97）
  }
  public static void sayHello(long args){
    System.out.println("hello long");  //优先级三，进一步转型为长整数97L，（安装char->int->long->float->double的顺序转型）
  }
  public static void sayHello(Character args){
    System.out.println("hello Character"); //优先级四，发生一次自动装箱，'a'被包装成它的封装类型java.lang.Character
  }
  public static void sayHello(char args){
    System.out.println("hello char");  //优先级一（最高），‘a’是一个char类型的数据
  }
  public static void sayHello(char... args){
    System.out.println("hello char..."); //优先级七，变长参数的重载优先级是最低的，这时候字符'a'被当做了一个数组元素
  }
  public static void sayHello(Serializable args){
    System.out.println("hello Serializable"); //优先级五，java.lang.Serializable是java.lang.Character类实现的一个接口，当自动装箱之后发现还是找不到装箱类，但是找到了装箱类实现的接口类型，所以又发生了一次自动转型。char可以转型成int，但Character是绝对不会转型为Integer的，它只能安全地转型为它实现的接口或父类。
  }
  public static void main(String[] args){
    sayHello('a');
  }
}
```

优先级规则：
1. 先char-int-long-float-double方向转换（byte和short不会，因为转型不安全）
2. 自动装箱（char-Character）
3. 检查接口
4. 检查父类
5. 变长参数

#### 动态分派——重写(Override)的体现

* 重写：两同两小一大
    * 方法名、形参列表 **相同**
    * 子类返回类型、抛出异常 **小于等于** 父类方法返回类型、父类方法抛出异常
    * 子类访问权限 **大于等于** 父类方法访问权限。
  
```java
public class DynamicDispatch{
  static abstract class Humam{
    protected abstract void sayHello();
  }
  static class Man extends Human{
    @override
    protected void sayHello(){
      System.out.println("man say hello");
    }
  }
  static class Woman extends Human{
    @override
    protected void sayHello(){
      System.out.println("woman say hello");
    }
  }
  public static void main(String[] args){
    Human man = new Man();
    Human woman = new Woman();
    man.sayHello();
    woman.sayHello();
    man = new Woman();
    man.sayHello();
  }
}
/**
 *运行结果：
 *man say hello
 *woman say hello
 *woman say hello
 **/
```

* invokevirtual指令的运行时解析过程大致分为以下几个步骤：
    * 找到操作数栈顶的第一个元素所指向的对象的 实际类型 ，记作C
    * 如果在类型C中找到与常量中的描述符和简单名称都相符的方法，则进行访问权限校验，如果通过则返回这个方法的直接引用，查找结束；如果不通过，则返回 `java.lang.IllegalAccessError` 异常。
    * 否则，按照继承关系从下往上依次对C的各个父类进行第2步的搜索和验证过程。
    * 如果始终没有找到合适的方法，则抛出 `java.lang.AbstractMethodError` 异常。
* 由于invokevirtual指令执行的第一步就是在运行期确定接收者的 **实际类型** ，所有两次调用中的invokevirtual指令把常量池中的类方法符号引用解析到了不同的直接引用上，这个过程就是Java语言中 **方法重写的本质** 。

* 我们把这种在 **运行期** 根据实际类型确定方法执行版本的分派过程称为动态分派。

### 单分派与多分派

```java
public class Dispatch {
    static class QQ {}
    static class _360 {}
    public static class Father {
        public void hardChoice(QQ arg) {
            System.out.println("father choose qq");
        }
        public void hardChoice(_360 arg) {
            System.out.println("father choose 360");
        }
    }
    public static class Son extends Father {
        public void hardChoice (QQ arg) {
            System.out.println("son choose qq");
        }
        public void hardChoice (_360 arg) {
            System.out.println("son choose 360");
        }
    }
    public static void main(String[] args) {
        Father father = new Father();
        Father son = new Son();
        father.hardChoice(new _360());
        son.hardChoice(new QQ());
    }
}
//father choose 360
//son choose qq
```

* 方法的接收者与方法的参数统称为方法的 **宗量** 。单分派是根据一个宗量对目标方法进行选择，多分派则是根据多于一个宗量对目标方法进行选择。我们看编译阶段编译器的选择过程，也就是静态分派的过程。这时选择目标方法的依据有两点：一是静态类型是Father还是Son，二是方法参数是QQ还是360这次选择结果产生了两条invokevirtual指令，两条指令的参数分别是常量池中指向 `Father.hardChoice(360)` 及 `Father.hardChoice(QQ)` 方法的符号引用。因为是根据两个宗量进行选择，所以Java语言的静态分派属于多分派类型。

* 再看运行阶段虚拟机的选择，也就是动态分派的过程。在执行 `son.hardChoice(new QQ())` 这句代码时（准确地说时在执行这句代码所对应的invokevirtual指令时），由于 编译期已经决定目标方法的签名必须是 `hardChoice(QQ)` ，虚拟机此时不会关心传递过来的参数“QQ”到底时“腾讯QQ”还是“奇瑞QQ”，因为这时参数的静态类型、实际类型都对方法的选择 不会 构成影响，唯一可以影响虚拟机选择的因素只有 此方法的接受者的实际类型 是Father还是Son。因为只有一个宗量作为选择依据，所以Java语言的动态分派属于单分派类型。

* Java语言的 **静态分派** 属于 **多分派** 类型； **动态分派** 属于 **单分派** 类型。

### 虚拟机动态分派的实现

* “稳定优化” 手段就是为类在方法区中建立一个「虚方法表」（Vritual Methdo Table，也成为vtable，与此对应的，在invokeinterface执行时也会用到接口方法表——Interface Method Table，简称itable），使用虚方法表索引来代替元数据查找以提高性能。
* 虚方法表中存放着各个方法的实际入口地址。
  * 如果某个方法在子类中没有被重写，那子类的虚方法表里面的地址入口和父类相同方法的地址入口是一致的，都指向父类的实现入口。
  * 如果子类中重写了这个方法，子类方法表中的地址将会替换为指向子类实现版本的入口地址。

## 基于栈的字节码解释执行引擎

### 解释执行

[![jvm-tu-8.4.png](https://i.loli.net/2018/08/14/5b728be07937e.png)](https://i.loli.net/2018/08/14/5b728be07937e.png)

* 词法分析、语法分析以至后面的优化器和目标代码生成器都可以选择独立与执行引擎，形成一个完整意义的编译器去实现，这类代表是 c/c++ 语言。
* 选择把其中的一部分步骤（如生成抽象语法树之前的步骤）实现为一个半独立的编译器，这类代表是 Java 语言。
* 把这些步骤和执行引擎全部集中封装在一个封闭的黑匣子之中，如大多数的 JavaScript 。
* Java 语言中，Javac编译器完成了程序代码经过词法分析、语法分析到抽象语法树，再遍历语法树生成线性的字节码指令流的过程。因为这一部分动作是在Java虚拟机之外进行的，而解释器在虚拟机的内部，所以Java的编译解释 半独立 的实现。


### 基于栈的指令集与基于寄存器的指令集

* 栈指令集：
  * 优点：可移植，代码更加紧凑，编译器实现简单
  * 确定：速度慢，完成相同功能指令数量多

### 基于栈的解释器执行过程

具体见书
