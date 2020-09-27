- AQS：AbstractQueuedSychronizer同步发生器，构建LOCK。在JUC包ReentrantLock、读写锁、信号量的基础都是AQS
  - AQS架构组成
    ![](/assets/iShot2020-09-27下午02.06.12.png)
    - AQS中队列只有两个：同步队列和条件队列，底层数据结构都是链表
    - 图中颜色代表四种不同场景，序号代表看的顺序
  - AQS本身就是一套锁的框架，它定义了获得锁和释放锁的代码结构，所以如果要新建锁，只要继承AQS，并实现相应方法即可。
  - 通过内置得到FIFO同步队列来完成线程争夺资源的管理工作
  - 同步器来管理 需要获取资源的Node线程队列，每个线程做的事情就是获取锁、释放锁，然后队列里的线程通过自旋锁去公平竞争资源

- 基本属性
  - 简单属性
    - `private volatile int state;` <br>
    同步器的状态，子类会根据状态字段进行判断是否可以获得锁；比如CAS成功给state赋值1算得到锁，CAS成功给state赋值算释放锁；可重入锁，每次获得锁+1，每次释放锁-1。最重要的就是 state 属性，是 int 属性的，所有继承 AQS 的锁都是通过这个字段来判断能不能 获得锁，能不能释放锁。
    - `static final long spinForTimeoutThreshold = 1000L;`<br>
    自旋超时阈值，单位纳秒，当设置等待时间时，才会用到这个属性。
    
  - 同步队列
    - 同步队列的主要作用阻塞获取不到锁的线程，并在适当时机释放这些线程。同步队列底层数据结构是一个双向链表。
    - `private transient volatile Node head;` 同步队列链表头
    - `private transient volatile Node tail;` 同步链表的尾
