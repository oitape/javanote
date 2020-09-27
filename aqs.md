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

  - 条件队列属性
    - 和同步队列一样管理获取不到锁的线程，底层数据结构也是链表，但条件队列不直接和锁打交道，常常和锁配合使用，是一定场景下对锁功能的一种补充。

    ```java
    // 条件队列，从属性上可以看出是链表结构
    public class ConditionObject implements Condition, java.io.Serializable { 
      private static final long serialVersionUID = 1173984872572414699L; 
      // 条件队列中第一个 node
      private transient Node firstWaiter;
      // 条件队列中最后一个 node  
      private transient Node lastWaiter; 
    }
    ```
    - ConditionObject 是实现 Condition 接口的，Condition 接口相当于 Object 的各种监控方 法，比如 Object#wait ()、Object#notify、Object#notifyAll 这些方法
    - 共享锁和排它锁区别？
      - 共享锁：允许多个线程获得同一个锁，并且可以设置获取锁的线程数量
      - 排它锁：同一个时刻只有一个线程可以获取锁
    - 获取锁
      - 比如使用Lock.lock()，Lock一般是AQS的子类，lock方法一般会调用AQS的acquire或tryAcquire方法，acquire方法AQS已经实现了，tryAcquire需要子类实现，
    - 释放锁
      - 比如Lock.unLock ()方法，目的是让线程释放对资源的访问权
    - acquire（）和 release（）方法时AQS提供的一种框架流程，实际需要子类去实现tryAcquire（）和tryRelease（）方法
      
* 自定义锁

  ```java
  public class MyLock implements Lock {
    private Helper helper = new Helper();
    private class Helper extends AbstractQueuedSynchronizer {
        // 获取锁
        @Override
        protected boolean tryAcquire(int arg) {
            int state = getState();
            // 拿到锁
            if (state == 0) {
                // 利用CAS原理修改state
                if (compareAndSetState(0, arg)) {
                    // 设置当前锁占用资源
                    setExclusiveOwnerThread(Thread.currentThread());
                    return true;
                }
            } else if (getExclusiveOwnerThread() == Thread.currentThread()) { //锁重入性
                setState(getState() + arg);
                return true;
            }
            return false;
        }

        //释放锁
        @Override
        protected boolean tryRelease(int arg) {
            int state = getState() - arg;
            boolean flag = false;
            // 判断释放后是否为0
            if (state == 0) {
                setExclusiveOwnerThread(null);
                setState(state);
                return true;
            }
            setState(state);
            return false;
        }

        public Condition newConditionObject() {
            return new ConditionObject();
        }
    }

    @Override
    public void lock() {
        helper.acquire(1);
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {
        helper.acquireInterruptibly(1);
    }

    @Override
    public boolean tryLock() {
        return helper.tryAcquire(1);
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return helper.tryAcquireSharedNanos(1, unit.toNanos(time));
    }

    @Override
    public void unlock() {
        helper.release(1);
    }

    @Override
    public Condition newCondition() {
        return helper.newConditionObject();
    }
  }
  ```

