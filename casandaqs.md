- 锁
    - 悲观锁：写多，读少。采用LOCK
    - 乐观锁：写少，读多。采用版本机制。每写一次版本增加
- CAS：CompareAndSwap
    - 一种无锁的原子算法，乐观锁。给一个期望值，与现有的值比较，如果相等再修改，不相等什么都不做
    CAS(V,E,N)：V是拿到的值，E期望的值，N要修改的值。如果V=E才会将V的值修改成N。如果V!=E则说明已经被人修改，不做操作
    - CAS实现稍微复杂，无锁，不存在阻塞，提高效率，CPU的吞吐量。
    - CAS是靠硬件实现的，从而在硬件层面提升效率。
    - JUC下的atomic类都是通过CAS来实现的
    
- CAS缺点
    - 循环时间太长：如果CAS一直不成功呢？这种情况绝对有可能发生，如果自旋CAS长时间地不成功，则会给CPU带来非常大的开销。在JUC中有些地方就限制了CAS自旋的次数，例如BlockingQueue的SynchronousQueue。
    - 只能保证一个共享变量原子操作：看了CAS的实现就知道这只能针对一个共享变量，如果是多个共享变量就只能使用锁了，当然如果你有办法把多个变量整成一个变量，利用CAS也不错。例如读写锁中state的高地位
    - ABA问题：CAS需要检查操作值有没有发生改变，如果没有发生改变则更新。但是存在这样一种情况：如果一个值原来是A，变成了B，然后又变成了A，那么在CAS检查的时候会发现没有改变，但是实质上它已经发生了改变，这就是所谓的ABA问题。对于ABA问题其解决方案是加上版本号，即在每个变量都加上一个版本号，每次改变时加1，即A —> B —> A，变成1A —> 2B —> 3A。
- thread.Join()：把该线程加入到当前线程，比如main中走到thread.Join()时会先执行thread完毕后再执行main。多线程使用join可以使线程按照自己指定的顺序执行


- AQS：AbstractQueuedSychronizer同步发生器，构建LOCK。在JUC包ReentrantLock、读写锁、信号量的基础都是AQS    
    - 通过内置得到FIFO同步队列来完成线程争夺资源的管理工作
    - 同步器来管理 需要获取资源的Node线程队列，每个线程做的事情就是获取锁、释放锁，然后队列里的线程通过自旋锁去公平竞争资源

- 自定义锁
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
                // 利用CAS原理又该state
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

- CountDownLatch：等所有线程都执行完了才继续下一步
    ```java
        CountDownLatch latch=new CountDownLatch(10);
        for (int i = 0; i < 10; i++) {
            threads[i]=new Thread(()->{
                int val=new Random().nextInt(10);
                try {
                    TimeUnit.SECONDS.sleep(val);
                    latch.countDown();
                } catch (InterruptedException e) {
                    latch.countDown();
                }
            });
            threads[i].start();
        }
        latch.await();
        System.out.println("唤醒主线程")
    ```
- CyclicBarrier：用于所有线程都准备好了再同时执行
    ```java
        CyclicBarrier barrier=new CyclicBarrier(8);
        Thread[] play=new Thread[8];
        for (int i = 0; i < 8; i++) {
            play[i]=new Thread(()->{
                try {
                    TimeUnit.SECONDS.sleep(new Random().nextInt(10));
                    System.out.println(Thread.currentThread().getName()+"准备好了");
                    barrier.await();
                } catch (Exception e) {
                    e.printStackTrace();
                }
                // 等到8个线程全部准备好了  才统一一起往下走
                System.out.println("选手"+Thread.currentThread().getName()+"起跑");
            },"play["+i+"]");
            play[i].start();
        }
    ```
    
- Semaphore：用于控制同时运行的总数
    ```java
        Semaphore semaphore = new Semaphore(5);
        Thread[] threads = new Thread[10];
        for (int i = 0; i < threads.length; i++) {
            int finalI = i;
            threads[i] = new Thread(() -> {
                try {
                    TimeUnit.SECONDS.sleep(2);
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + " 可以入场");
                } catch (Exception e) {

                }
                try {
                    TimeUnit.SECONDS.sleep(finalI);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                semaphore.release();
                System.out.println(Thread.currentThread().getName() + " 离场");
            });
            threads[i].start();
        }
    ```
- JUC
    - 非阻塞式集合：ConcurrentLinkedDeque
        包含添加移除方法：方法不能立即被执行，则返回null或抛出异常，但是调用这个方法的线程不会被阻塞
    - 阻塞式集合：LinkedBlockingDeque
        当集合已满或为空时，被调用的添加或者移除方法就不能立即被执行，那么调用这个方法的线程将被阻塞，一直到该方法可以被成功执行
    
    
    
    