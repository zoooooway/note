# JVM
> https://docs.oracle.com/javase/specs/jvms/se8/html/index.html

> todo: 使用 go 实现一个 JVM。

## JVM 结构

### 运行时数据区（Run-Time Data Areas）

#### 程序计数器（PC Register） 
每个 Java 虚拟机线程都有自己**私有**的程序计数器。
在任何时候，每个 Java 虚拟机线程都在执行该线程当前方法的代码，而程序计数器的值则指向当前正在执行的 Java 虚拟机指令的地址

#### Java 虚拟机栈（Java Virtual Machine Stacks）
Java 虚拟机栈用于执行方法。
每个 Java 虚拟机线程都有一个**私有**的 Java 虚拟机栈，与 Java 虚拟机线程同时创建。 Java 虚拟机栈存储栈帧（Frames）。


#### 堆（Heap）
Java 虚拟机有一个在所有 Java 虚拟机线程之间**共享**的堆。堆是运行时数据区域，所有类实例的内存和数组的内存都从这里分配。

#### 方法区（Method Area）
Java 虚拟机有一个在所有 Java 虚拟机线程之间**共享**的方法区。
它存储每个类的**结构**，例如**运行时常量池**、**字段**和**方法数据**，以及**方法和构造函数的代码**，包括类和实例初始化以及接口初始化中使用的特殊方法

##### 运行时常量池（Run-Time Constant Pool）
运行时常量池是每个类或每个接口的 class 文件中 `constant_pool` 表的运行时表示形式。

**每个运行时常量池都是从 Java 虚拟机的方法区分配的**。类或接口的运行时常量池是在 Java 虚拟机创建类或接口时构造的。

#### 本地方法栈（Native Method Stacks）
与 Java 虚拟机栈类似，区别是用来执行本地方法（Native Method）的。通常在创建每个Java 虚拟机线程时为每个Java 虚拟机线程分配本机方法栈。