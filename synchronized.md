- 获取对象锁
  - synchronized(this|object) {}
  - 修饰非静态方法
  - 在 Java 中，每个对象都会有一个 monitor 对象，这个对象其实就是 Java 对象的锁，通常会被称为“内置锁”或“对象锁”。类的对象可以有多个，所以每个对象有其独立的对象锁，互不干扰。
- 获取类锁
  - synchronized(类.class) {}
  - 修饰静态方法
  - 在 Java 中，针对每个类也有一个锁，可以称为“类锁”，类锁实际上是通过对象锁实现的，即类的 Class 对象锁。每个类只有一个 Class 对象，所以每个类只有一个类锁。


- Java中，每个对象都会有一个monitor对象，监视器
  - 某一个线程要占有这个对象的时候，先monitor的计数器是不是0，0代表无线程占有，占有该对象时，monitor+1；不为0，则有其他线程已经占有；线程释放时，monitor-1.
  - 同一线程可以对同一对象进行多次加锁，+1，+1，重入性
  
  
- synchronized底层原理
通过`javap -v $className` 可查看
  - 方法内synchronized可以看到
    - monitorenter 和 monitorexit 
  - synchronized 静态方法
    ```java
    public static synchronized void accessResources0();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_STATIC, ACC_SYNCHRONIZED
    ```
    - 可以看到通过ACC_SYNCHRONIZED flag进行加锁实现同步静态方法
  
- Java虚拟机对synchronized的优化
  - 偏向锁 （jdk1.6之后）
  - 轻量级锁（jdk1.6之后）
  - 重量级锁 
  
  一个对象实例包含：对象头、实例变量、填充数据
  ![](/assets/instance.png)
  - 对象头：加锁的基础
  ![](/assets/instance-header.png)
     - Mark Word中 锁信息
     ![](/assets/lock-info.png)
     - 无锁状态:没有加锁
     - 偏向锁：在对象第一次被某一线程占有的时候，是否偏向锁置1，锁标记01，写入线程号；当其他的线程访问的时候，竞争，可能失败也可能成功。倾向被第一次占有它的线程获取次数多，成功
     - CAS算法
     - 自旋锁：竞争失败的时候，不是马上转化级别，而是执行几次空循环5 10 
     - 锁消除：JIT在编译的时候吧不必要的锁去掉
  - 实例变量
  - 填充数据：8字节


  
  
  
  
  
  
  