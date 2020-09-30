- ThreadLocal
    - ThreadLoal是线程局部变量，和线程一对一
    - ThreadLocal 变量通常被private static修饰。线程结束时，对应的ThreadLocal 的实例副本都可被回收
    - ThreadLocal 适用于每个线程需要自己独立的实例且该实例需要在多个方法中被使用，也即变量在线程间隔离而在方法或类间共享的场景。
    - 实现原理：
        - ThreadLocal内部类ThreadLocalMap，set()、get()方法都是调用这个内部类的方法，ThreadLocal只是ThreadLocalMap的封装，传递了变量值。
    - 内存泄露问题
        - 弱引用在下次YGC的时候就会被回收
        - ThreadLocalMap中的Entry是使用ThreadLocal的弱引用作为key，一个ThreadLocal不存在外部强引用时，Key(ThreadLocal)势必会被GC回收，这样就会导致ThreadLocalMap中key为null， 而value还存在着强引用，只有thead线程退出以后,value的强引用链条才会断掉
        - 当key为null，在下一次ThreadLocalMap调用set(),get()，remove()方法的时候会被清除value值。
        
        - 解决办法：<br> a、每次使用完ThreadLocal都调用它的remove()方法清除数据；<br>b、将ThreadLocal变量定义成private static，这样就一直存在ThreadLocal的强引用，也就能保证任何时候都能通过ThreadLocal的弱引用访问到Entry的value值，进而清除掉 。

- ThreadLocalMap
    - Map 结构，底层是数组，有初始化大 小，也有扩容阈值大小，数组的元素是 Entry，Entry 的 key 就是 ThreadLocal 的引用，value 是 ThreadLocal 的值。
    
- ThreadLocal 是如何做到线程之间数据隔离的?
    - 通过查看Thread类的源码发现
    ```java
        /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;

    /*
     * InheritableThreadLocal values pertaining to this thread. This map is
     * maintained by the InheritableThreadLocal class.
     */
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
    ```
    ThreadLocalMap是Thread类的属性，所以每个线程的ThreadLocals都是隔离独享的。一个线程同一时刻只能对ThreadLocalMap进行操作，因为同一个线程执行业务逻辑必然是串行的，那么操作ThreadLocalMap也必然是串行。
    - 此外在线程初始化的时候也会把父线程inheritThreadLocals进行传递
    ```java
    if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
            ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
    ```
    
    
    