- Thread
    - 每个线程都有优先级，高优先级的线程可能会优先执行
    - 父线程创建子线程后，优先级、是否是守护线程等属性父子线程是一致的
    - JVM启动时，通常都启动MAIN非守护线程，以下情况发生，线程就会停止：退出方法被调用（Thread.interrupt方法）；所有非守护线程都消亡或者从运行的方法正常返回，或者运行的方法抛出异常
    
- 线程状态
    ![](/assets/iShot2020-09-17下午03.47.05.png)

- 守护线程
    - 默认创建的线程都是非守护线程，创建守护线程时，需要将Thread的daemon属性设置为true，守护线程的优先级很低，当JVM退出时，是不关心有无守护线程的，即使还有很多守护线程，JVM仍然会退出。工作中，我们可能会实现一些监控工作，这时用守护线程去做，这样即使监控抛出异常，但因为是子线程，所以不会影响到主业务线程，因为是守护线程，JVM也无需关心守护线程，该退出就退出。

- ClassLoader：类加载器，把类文件、二进制数组、URL等位置加载成可运行class

- 创建线程的三种方法
    - 继承Thread
    - 新建类实现runnable接口，作为入参
    - 匿名内部类实现
    
- join：当前线程等待另一个线程执行完成之后，才能继续操作
    ```java
Thread main = Thread.currentThread(); log.info("{} is run。",main.getName());
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        log.info("{} begin run",Thread.currentThread().getName()); 
    try {
        Thread.sleep(30000L);
    } catch (InterruptedException e) {
        e.printStackTrace(); }
        log.info("{} end run",Thread.currentThread().getName()); }
    });
// 开一个子线程去执行
thread.start();
// 当前主线程等待子线程执行完成之后再执行 
thread.join();
    ```
    
- yield: 是一个native方法
    - 当前线程做出让步，放弃当前cpu，让cpu重新选择线程，避免线程过度使用cpu，我们在写while死循环的时候，预计短时间内，while可以结束的话，可以在死循环里使用yield方法，防止cpu一直while死循环霸占

- sleep：是一个native方法
    - 当前线程沉睡多久，沉睡期间不会释放锁资源，其他线程是无法得到锁的

- interrupt：可以打断终止正在运行的线程
    - Object#wait ()、Thread#join ()、Thread#sleep (long) 这些方法运行后，线程的状态是 WAITING 或 TIMED_WAITING，这时候打断这些线程，就会抛出 InterruptedException 异常，使线程的状态直接到 TERMINATED;
    - 如果 I/O 操作被阻塞了，我们主动打断当前线程，连接会被关闭，并抛出 ClosedByInterruptException 异常;
    
    
- 面试题
    - 创建子线程时，子线程得不到父类的ThreadLocal，有什么办法可以解决这个问题？
     - 可以使用InheritableThreadLocal来代替ThreadLocal，两者都是线程的属性，所以可以做到线程之间的数据隔离，父线程 ThreadLocal 是无法传递给子线程 的，但 InheritableThreadLocal 可以。
     

