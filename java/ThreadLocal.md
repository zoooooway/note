# ThreadLocal
> https://learn.lianglianglee.com/专栏/Java 并发编程 78 讲-完/46 多个 ThreadLocal 在 Thread 中的 threadlocals 里是怎么存储的？.md

`ThreadLocal` 类提供每个线程的 "私有" 局部变量, 这里的 "私有" 并不是指可见性修饰符 `private` 的含义, 而是指这个局部变量是每个 `Thread` 实例都有其属于自己的一份.
这意味着每个线程都可以通过 `ThreadLocal` 来存储与当前线程相关联的变量, 这一特点使得 `ThreadLocal` 通常用来在线程中传递上下文.

`Thread` 类是 `Java` 中对应操作系统线程的抽象, 通常, 一个 `Thread` 实例即对应一个操作系统的线程. 
每个 `Thread` 实例其内部持有一个 `ThreadLocalMap` 变量: 
```java
public class Thread implements Runnable {
    // ...

    /* 
     * ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. 
     */
    ThreadLocal.ThreadLocalMap threadLocals = null;

    // ...
}
```

`ThreadLocalMap` 是一个类似 `HashMap` 的数据结构, 它以 key => value 的形式存储数据. 内部结构如下:
```java
static class ThreadLocalMap {
    
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }

    /**
    * The table, resized as necessary.
    * table.length MUST always be a power of two.
    */
    private Entry[] table;

    // ...
}
```
可以看到 `ThreadLocalMap` 内部和 `HashMap` 一样, 使用一个数组存储值. 其存储的值是 `Entry` 类型, `Entry` 继承了 `WeakReference`, 这表明 `Entry` 的实例是弱引用. 这样做的作用是帮助 `GC`, 避免内存泄露. 但即使使用弱引用, 使用 `ThreadLocal` 时仍然可能出现内存泄露.

`Thread`, `ThreadLocal` 以及 `ThreadLocalMap` 的关系如下图:

![](https://raw.githubusercontent.com/zoooooway/picgo/nom/202304231153762.png)

每个线程中都持有一个属于自己的 `ThreadLocalMap` 实例, `ThreadLocalMap` 中的键是 `ThreadLocal`. 因为每个线程的 `ThreadLocalMap` 都是不同的, 所以通过 `ThreadLocal` 去获取对应的 `value` 时所得到的值也是不同的.

## 内存泄露
`ThreadLocal` 被人所熟知的一个使用问题就是内存泄露. 

首先, 先要理解为什么 `Entry` 会是弱引用类型的?
先假设 `Entry` 并不是弱引用类型. 如果我们不需要使用 `ThreadLocal` 变量, 会通过语句 `ThreadLocal tl = null` 来将 `ThreadLocal` 变量变为可回收对象, 通过 `GC` 来将这个变量的内存回收掉. 但 `Entry` 中是引用了 `ThreadLocal` 这一变量的, 因此 `GC` 并不会回收它. 那么需要 `Entry` 也变成可回收对象, 但 `Entry` 是存在 `ThreadLocalMap` 中的 , `ThreadLocalMap` 是被 `Thread` 实例所引用的.

![](https://raw.githubusercontent.com/zoooooway/picgo/nom/202304272123530.png)

因此, 只要 `Thread` 还存活, 那么 `ThreadLocalMap` 就不会被回收. 这意味着单单通过将 `ThreadLocal` 的值置为 `null` 是没办法回收 `ThreadLocal` 对象的, 因为在 `ThreadLocalMap` 中仍然存在 `ThreadLocal` 的引用. 由于大多数情况下, 我们都会使用线程池, 池中线程往往不会被销毁, 那么除非手动将 `ThreadLocalMap` 中的 `Entry` 清除, 否则 `ThreadLocal` 一直不会被回收.

在前面提到了, `ThreadLocalMap` 中的值是 `Entry` 类型, `Entry` 的实例是弱引用. 弱引用意味着: 假如对象 A 只被弱引用的对象 B 所引用，那么 A 可以被回收，即弱引用的对象不会阻止 `GC`.
在创建一个 `Entry` 实例时, 构造函数需要两个值: `ThreadLocal`类型的 k 和要存储的实际值 v, 构造函数中声明 `Entry` 弱引用 k，这表示如果 k 的实例只有 `Entry` 引用它, 没有任何强引用了的时候, 那么 `JVM` 会将 K 回收. 
现在同样的, 通过语句 `ThreadLocal tl = null` 来将 `ThreadLocal` 变量变为可回收对象. 同样的, `Entry` 中引用了 `ThreadLocal` 这一变量, 但由于 `Entry` 是弱引用, 如果 k 只有在 `Entry` 中被引用, 那么 `JVM` 仍然会回收它. 这样就避免了 `ThreadLocal` 在 `Entry` 中被强引用而去阻止 `GC`.

![](https://raw.githubusercontent.com/zoooooway/picgo/nom/202304272125055.png)

上述可以总结为一点: `Entry` 使用弱引用来使得 `JVM` 在 `ThreadLocal` 不使用时(不可达)回收 `ThreadLocal` 对象. 
这句话的重点是回收 `ThreadLocal` 对象. 在 `ThreadLocalMap` 中, `Entry` 是散列表中存储的条目，每一个 `Entry` 包含一个 `ThreadLocal` 对象的键和真正存储的变量 `value`. 注意上述 `Entry` 构造函数中的这个语句: `value = v;`, 这表明 `value` 强引用了传入的参数 `v`, 这意味着只要 `Entry` 不被回收, 对应的对象 `v` 也不会被回收. 
`JDK` 考虑到这一点, 因此在 `ThreadLocal` 的实现里, 调用 `set`, `remove`, `rehash` 这些方法时, 会去清除过期的 `Entry`, 也就是 key 为 `null` 的 `Entry`.
但很可惜, 假如 `ThreadLocal` 不再使用了, 就像上面提到的, 已经置为 `null` 了. 那么也不会再调用这些方法了, 过期的 `Entry` 也不会被清除, 对象 `v` 也不会被回收了, 而这时就发生了内存泄露.

![](https://raw.githubusercontent.com/zoooooway/picgo/nom/202304272130966.png)

原理有些复杂, 但正确的做法很简单, 在需要清除 `ThreadLocal` 中包含的 `value` 时调用 `ThreadLocal` 的 `remove` 方法. 调用这个方法就可以删除对应的 `value` 对象, 这样就可以避免内存泄漏。







