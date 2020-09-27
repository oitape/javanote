* 锁
  * 悲观锁：写多，读少。采用LOCK
  * 乐观锁：写少，读多。采用版本机制。每写一次版本增加
* CAS：CompareAndSwap
  * 一种无锁的原子算法，乐观锁。给一个期望值，与现有的值比较，如果相等再修改，不相等什么都不做
    CAS\(V,E,N\)：V是拿到的值，E期望的值，N要修改的值。如果V=E才会将V的值修改成N。如果V!=E则说明已经被人修改，不做操作
  * CAS实现稍微复杂，无锁，不存在阻塞，提高效率，CPU的吞吐量。
  * CAS是靠硬件实现的，从而在硬件层面提升效率。
  * JUC下的atomic类都是通过CAS来实现的


* CAS缺点
  * 循环时间太长：如果CAS一直不成功呢？这种情况绝对有可能发生，如果自旋CAS长时间地不成功，则会给CPU带来非常大的开销。在JUC中有些地方就限制了CAS自旋的次数，例如BlockingQueue的SynchronousQueue。
  * 只能保证一个共享变量原子操作：看了CAS的实现就知道这只能针对一个共享变量，如果是多个共享变量就只能使用锁了，当然如果你有办法把多个变量整成一个变量，利用CAS也不错。例如读写锁中state的高地位
  * ABA问题：CAS需要检查操作值有没有发生改变，如果没有发生改变则更新。但是存在这样一种情况：如果一个值原来是A，变成了B，然后又变成了A，那么在CAS检查的时候会发现没有改变，但是实质上它已经发生了改变，这就是所谓的ABA问题。对于ABA问题其解决方案是加上版本号，即在每个变量都加上一个版本号，每次改变时加1，即A —&gt; B —&gt; A，变成1A —&gt; 2B —&gt; 3A。

* thread.Join\(\)：把该线程加入到当前线程，比如main中走到thread.Join\(\)时会先执行thread完毕后再执行main。多线程使用join可以使线程按照自己指定的顺序执行

- ReentrantLock
    - 可重入互斥锁：同一个线程可以对同一个共享资源重复的加锁或释放锁，互斥就是AQS中的排它锁的意思，只允许一个线程获得锁
    - 和synchronized锁具有同样的功能，但是更有扩展性
    - 构造器接收fairness参数，true保证获得锁时的顺序，可以保证同步队列中的线程从头到尾的顺序依次获得锁，false不保证
    - ReentrantLock本身是不继承AQS，实现了Lock接口
    
* CountDownLatch：等所有线程都执行完了才继续下一步

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

* CyclicBarrier：用于所有线程都准备好了再同时执行

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

* Semaphore：用于控制同时运行的总数

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

* JUC
  * 非阻塞式集合：ConcurrentLinkedDeque
      包含添加移除方法：方法不能立即被执行，则返回null或抛出异常，但是调用这个方法的线程不会被阻塞
  * 阻塞式集合：LinkedBlockingDeque
      当集合已满或为空时，被调用的添加或者移除方法就不能立即被执行，那么调用这个方法的线程将被阻塞，一直到该方法可以被成功执行



