# JVM





# 理论知识

## 基本概念

### 什么是JVM

- `Java virtual machine`- Java虚拟机

- 本质是一个运行在计算机上的程序，职责是运行`Java`字节码文件

![a024ccf0-bc10-4a16-aca9-e948173e700d](file:///C:/Users/flypiggy/Pictures/Typedown/a024ccf0-bc10-4a16-aca9-e948173e700d.png)



### 功能

- **解释和运行**：对于字节码文件中的指令，实时的解释成机器码，让计算机执行

- **内存管理**：自动为对象、方法等分配内存；自动的垃圾回收机制，回收不再使用的对象

- **即时编译**：对热点代码进行优化，提升执行效率
  *Java与c/cpp代码执行过程区别*
  ![305d2e38-6a7d-4604-ac19-62e3aab99c28](file:///C:/Users/flypiggy/Pictures/Typedown/305d2e38-6a7d-4604-ac19-62e3aab99c28.png)
  *Java需要实时解释，主要是为了支持跨平台特性*：字节码指令经过`JVM`解释可以在`win`/`Linux`等不同平台运行   
  *即时编译原理*：在程序运行时监测热点代码，达到一定条件后将其编译为机器码并缓存，以提高程序执行效率。

## 字节码文件详解

### Java虚拟机的组成

- 类加载器`ClassLoader`：加载`class`字节码文件中的内容到内存中

- 运行时数据区域（`JVM`管理的内存）：负责管理`JVM`使用到的内存，比如创建对象和销毁对象

- 执行引擎（即时编译器、解释器、垃圾回收器等）：将字节码文件中的指令解释成机器码，同时使用即时编译器优化性能

- 本地接口：调用本地已经编译的方法，比如虚拟机中提供`c/cpp`方法

### 字节码文件的组成

#### 概览

- 基础信息：魔数、字节码文件对应的`Java`版本号访问标识（`public、int`等），父类和接口

- 常量池：保存了字符串常量、类、接口名、字段名，主要在字节码指令中使用

- 字段：当前类或接口声明的字段信息

- 方法：当前类或接口声明的方法信息、字节码指令

- 属性：类的属性

#### 基本信息

 **Magic魔数**：`Java`字节码文件中，将文件头称为`magic`魔数，字节数是4，内容为`CAFEBABE`

**版本号**：编译字节码文件的`JDK`版本 

#### 常量池

- 避免相同内容重复定义，节省空间

- 常量池中的数据都有一个编号，编号从1开始，在字段或者字节码指令中通过百年好可以快速的找到对应的数据

- 字节码指令中通过编号引用到常量池的过程称之为符号引用

#### 方法

- 字节码中的方法区域是存放**字节码指令**的核心位置，字节码指令的内容存放在方法的`Code`属性中

- 操作数栈是临时存放数据的地方，局部变量表是存放方法中局部变量的位置
  
  eg `i++`和`++i`的底层   
  
  `i++`：   
  
  ![e2b1e38f-92fa-41c3-9afa-d258db40e625](file:///C:/Users/flypiggy/Pictures/Typedown/e2b1e38f-92fa-41c3-9afa-d258db40e625.png)  
  
  `++i`：   
  
  ![52868636-dd2b-4c22-bcba-d00e47432c6e](file:///C:/Users/flypiggy/Pictures/Typedown/52868636-dd2b-4c22-bcba-d00e47432c6e.png)   
  
  **通过分析字节码指令发现**，`i++`先把$0$取出来放入临时的操作栈中，接下来对$i$进行加$1$，$i$变成了$1$，最后再及那个之前报错的临时值$0$放入$i$，最后$i$就变成了$0$

### 字节码常用工具

#### javap -v

`JDK`自带的反编译工具，可以通过控制台查看字节码文件的欸容。适合在服务器上查看字节码文件内容

#### jclasslib插件版本

有`Idea`插件版本，可以再代码编译后实时看到字节码文件内容，直接到插件市场安装继续行了

- 当修改代码之后，需要`Build/Recompile`才会更新字节码文件

#### arthas

`Alibaba`开源的`Java`诊断工具，通过全局视角实时查看应用`load`、内存、`gc`、线程的状态信息，并能在不修改应用代码情况下，对业务问题进行针对，提升线上问题排查效率 

*常用功能*：监控面板，查看字节码信息，方法监控，类的热部署，内存监控，垃圾回收监控，应用热点定位

## 类的声明周期

一个类加载、使用、卸载的整个过程

### 概述

加载-连接（验证-准备-解析）-初始化-使用-卸载

### 加载阶段

- 类加载器根据类的全限定名通过不同的渠道以二进制流的方式获取字节码信息

- 在加载完类之后，`Java`虚拟机会将字节码中的信息保存到方法区中

- 在方法区中生成一个`InstanceKlass`对象，保存类的所有信息

- `Java`虚拟机会在堆中生成一份与方法区中数据类似的`java.lang.Class`对象，用于在`Java`代码中获取类的信息以及存储静态字段的数据

### 连接阶段

- 验证：验证字节码信息是否满足《Java虚拟机规范》

- 准备：给静态变量分配内存并赋初始值（默认值）
  
  若为`final`设置的静态变量，会直接赋值（就类似于直接执行了赋值语句，而非默认值）

- 解析：将常量池中的符号引用替换成指向内存的直接引用



### 解析阶段

- 将常量池中发符号引用替换为直接引用

- 直接引用不使用编号，而是使用内存中的地址进行访问具体的数据

### 初始化阶段

- 执行静态代码块中的代码，并为静态变量赋值

- 执行字节码文件中`clinit`部分的字节码指令   

eg1

![421e7da1-2a74-4a01-92f2-bad5cf07c24d](file:///C:/Users/flypiggy/Pictures/Typedown/421e7da1-2a74-4a01-92f2-bad5cf07c24d.png)    

- 当程序运行时，`JVM`首先加载`Test1`类，加载过程的初始化阶段会执行静态代码块中的代码，打印D。

- 进入`main`方法后，首先打印A

- 接下来执行`new Test1`，创建`Test1`实例。在创建实例的时候，非静态代码块（即{}中内容）会先与构造方法执行，于是先打印C再打印D

- 再次创建实例，因为`Test1`以及加载过了，所以不再执行静态代码块，但构造方法和构造代码块会重复，于是打印C和D

**类加载时 JVM 会收集所有静态代码块和静态字段并执行，由 JVM 保证静态代码块仅执行一次**

**在初始化阶段，静态代码块和静态变量赋值语句按照他们在代码中出现的顺序执行**

## 类加载器

负责将类的字节码加载到 JVM 中，并将其转化为可以被 JVM 执行的 Java 类

### 分类

* **启动类加载器（Bootstrap Class Loader）**：在jvm底层源码中实现，是 JVM 中最顶层的类加载器，由 C++ 实现
* **扩展类加载器（Extension Class Loader）**：由 Java 语言实现，负责加载 Java 的扩展类库，如`javax.*`开头的类。它的父类加载器是启动类加载器，加载的类通常位于`%JAVA_HOME%/jre/lib/ext`目录下。
* **应用程序类加载器（Application Class Loader）**：也称为系统类加载器，是 ClassLoader 类的子类，负责加载应用程序的类路径（CLASSPATH）下的所有类。它是默认的类加载器，在没有指定其他类加载器的情况下，应用程序中的类都是由它来加载的
* **自定义类加载器（Custom Class Loader）**：开发人员可以根据自己的需求自定义类加载器，用于加载特定的类或实现特殊的加载逻辑。例如，从网络上加载类、对加密的字节码进行解密后再加载等

#### 启动类加载器

负责加载 Java 核心类库，如`java.lang`、`java.util`等。它加载的类是 JVM 运行的基础，通常位于`%JAVA_HOME%/jre/lib`目录下。

#### 扩展类/应用程序类加载器

`JDK`提供的，使用java编写的

![4ff4027e-298f-4138-bc44-86f5d48986ad](file:///C:/Users/flypiggy/Pictures/Typedown/4ff4027e-298f-4138-bc44-86f5d48986ad.png)    

### 双亲委派机制

`Java`虚拟机中有多个类加载器，双亲委派机制就是解决一个类由谁加载的问题

**自底向上查找是否加载过，再自顶向下进行加载**    

- 向上查找如果以及加载过，就直接返回`Class`对象

- 如果所有父类加载器都无法加载此类，则由当前加载器自己尝试加载

**优先级**：启动类Bootstrap->扩展类Extension->应用程序类Application

### 打破双亲委派机制

 

* **自定义类加载器**：继承`ClassLoader`类，重写`loadClass`方法。在`loadClass`方法中，可以根据具体需求，不遵循双亲委派的默认逻辑，直接加载类。例如，先判断类是否已经被加载，如果未被加载，则直接从指定的路径或数据源加载类，而不先委托给父类加载器

* **使用线程上下文类加载器**：线程上下文类加载器是 Java 中每个线程都有的一个类加载器。可以通过`Thread.currentThread().setContextClassLoader`方法设置线程上下文类加载器，然后在需要的时候，利用线程上下文类加载器来加载类。这种方式可以在不改变双亲委派机制整体结构的情况下，在特定的线程中实现类的特殊加载

## 运行时数据区域（JVM管理的内存）

**总览**：    

线程不共享

* 程序计数器
* Java 虚拟机栈
* 本地方法栈

 线程共享

* 方法区
* 堆

### 程序计数器

- 也叫PC寄存器，每个线程会通过程序计数器记录当前要执行的字节码指令的地址

- 不会出现内存溢出

### 栈

#### Java虚拟机栈

java虚拟机栈随着线程的创建而创建，在线程销毁是进行回收。由于方法可能会在不同的线程中执行，每个线程都会包含一个自己的虚拟机栈

##### 栈帧

**局部变量表**

- 这是一个数组结构，用于存放方法参数和方法内部定义的局部变量。

- 局部变量表的容量以变量槽（Variable Slot）为最小单位，每个变量槽可以存放一个`boolean`、`byte`、`char`、`short`、`int`、`float`、`reference`或者`returnAddress`类型的数据。对于`long`和`double`类型的数据，会使用两个连续的变量槽来存储

- 保存的内容：实例方法的this对象，方法的参数，方法体中声明的局部变量

- 槽是可以复用的，一旦某个局部变量不再生效（声明周期结束），当前槽就可以再次被使用

**操作数栈**

它是一个后进先出（LIFO）的栈结构，主要用于在方法执行过程中进行操作数的存储和运算。在方法执行时，操作数栈会根据字节码指令，将数据压入栈或者从栈中弹出。例如，在执行加法运算时，会先将两个操作数压入操作数栈，然后执行加法指令，将栈顶的两个操作数弹出进行相加，最后将结果压入栈中

**帧数据**

- 动态链接：当前类的字节码指令引用了其他类的属性或者方法时，需要将符号引用转换成对应的运行时常量池中的内存地址。动态链接就保存了编号到运行时常量池的内存地址的映射关系

- 方法出口：方法在正确或者异常结束时，当前栈帧会被弹出，同时程序计数器应该指向上一个栈帧中的下一条指令的地址。所以在当前栈帧中，需要存储此方法出口的地址

- 异常表：存放的是代码中异常的处理信息，包含了异常捕获的生效范围以及异常发生后跳转到的字节码指令的位置

##### 内存溢出

- 如果栈帧过多，占用内存超过栈内存可以分配的最大大小就会出现内存溢出

- 内存溢出时会出现StackOverflowError的错误

#### 本地方法栈

- 存储的是native本地方法的栈帧

### 堆

- 创建出来的对象都在堆上

- 堆内存会溢出，达到上限后抛出OutOfMemory的错误

- 堆空间三个值，used当前已使用的堆内存，total是java虚拟机已经分配的可用堆内存，max是java虚拟机可用分配的最大堆内存

**堆内存溢出**：gc讲

### 方法区

- 是存放基础信息的位置，线程共享，主要由三部分每个类的基本信息（元信息）、运行时常量池、字符串常量池

### 直接内存

- 不属于java运行时的内存区域

## 自动垃圾回收

- c/cpp中，一个对象如果不再使用，需要手动释放，否则会内存泄漏（不再使用的对象在系统中未被挥手，内存泄漏的积累可能导致内存溢出）

- java中引入了自动的**垃圾回收**（Garbage Collection简称GC）机制，通过垃圾回收器堆不再使用的对象完成自动的回收，垃圾回收器主要负责堆上的内存进行回收

### 方法区的回收

- 方法区中能回收的内容主要是不再使用的类   
  
  判定一个类可以被卸载，需要同时满足三个条件：此类所有实例对象都已被回收，在堆中不存在任何该类的实例对象以及子类对象；加载该类的类加载器以及被回收；该类队形的java.lang.Class对象没有在任何地方被引用

- 手动请求回收
  
  调用System.gc()方法，向Java虚拟机发送一个垃圾回收的请求，Java虚拟机会自行判断是否需要执行垃圾回收

### 堆回收

- 只有无法通过引用获取到对象时，该对象才能被回收

- 两种方法判断堆的对象是否被引用：引用计数法，可达性分析法

#### 引用计数法

- 为每个对象维护一个引用计数器，当对象被引用时加1，取消引用时减1

- 优点是实现简单，缺点是每次引用和取消引用都需要维护计数器，存在循环引用问题 

#### 可达性分析算法

- Java使用的是可达性分析算法来判断对象是否可回收。可达性分析将对象分为两类：垃圾回收的跟对象（GC Root）和普通对象，对象与对象之间存在引用关系

- 如果从某个GC Root对象能到的对象，则该对象不能被回收

##### GC Root对象

- 线程THread对象

- 系统类加载器加载的java.lang.Class对象

- 监视器对象，用来保存同步锁synchronized官架子持有的对象

- 本地方法调用时使用的全局对象
