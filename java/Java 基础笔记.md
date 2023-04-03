# Java 基础笔记

- ### static method and static fileds

  * 类可以从它的直接超类继承了超类的所有**具体**方法 (包括静态方法和实例方法)
  * 类可以从其直接超类和直接超接口继承所有的 abstract 和 default 方法 
  * 类不从其超接口继承静态方法
  * 允许实例变量**隐藏**静态变量

  > 参见：https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#d5e12110

* ### 方法区

  方法区与 Java 堆一样，是各个线程共享的内存区域，它用于存储**已被虚拟机加载的类信息**、**常量**、**静态变量**、**即时编译器编译后的代码**等数据。
