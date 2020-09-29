- 线程池
     ![](/assets/pool.png)
 - 核心线程数
 - 最大线程数
 - 线程保持活动的时间
 - 线程保持活动的时间单位
 - 任务采用的队列
 - 拒绝策略
    - newFixedThreadPool
        ```java
        new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
        ```
        - 固定大小的线程池，线程池中使用的队列是LinkedBlockingQueue，容量是默认的Integer的最大值，也就是线程池的处理能力有限时，阻塞队列中最大可以存放Integer最大值个任务，实际工作中，不建议使用newFixedThreadPool，因为其使用的队列默认构造器，容量太大了，在要求实时响应的请求中，队列容量太大也会有危害。比如调用方接口超时，但是服务端队列的任务还在排队执行，过了调用方的超时时间，服务端其实已经处理成功，但是调用方无法感知。
        
    - newCachedThreadPool
        ```java
        new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
        ```
        - 底层使用的SynchronousQueue，队列是没有大小限制的，请求多少队列都能承受住，缺点：每次put数据时并不能立马返回，需要等待有线程take之后才能正常返回，如果请求量大，而消费能力较差时，就会导致大量请求被holder住，必须等到慢慢消费完成之后才能被释放，所以使用该线程池也需要慎重。
    - newSingleThreadExecutor
        ```java
        new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
        ```
        - 底层也是采用的LinkedBlockingQueue，默认Integer最大值，这个线程池一次只能处理一个请求，其余请求都会在队列中排队执行。
    - newScheduledThreadPool
        ```java
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue());
        ```
        - 底层使用延迟队列，新的延迟请求都先到队列中，延迟时间到了线程池自然就能从队列中拿出线程进行执行
        
- 线程池的状态
  - RUNNING
      - 线程池创建后就处于Running状态
      - 并且线程池中的任务数为0
  - Shutdown
      - 线程池处在SHUTDOWN状态时，不接收新任务，但能处理已添加的任务。 
      - 调用线程池的shutdown()接口时，线程池由RUNNING -> SHUTDOWN。
  - Stop
      - 线程池处在STOP状态时，不接收新任务，不处理已添加的任务，并且会中断正在处理的任务。
      - 调用线程池的shutdownNow()接口时，线程池由(RUNNING or SHUTDOWN ) -> STOP。
  - TIDYING 
      - 当所有的任务已终止，任务数量”为0，线程池会变为TIDYING状态。当线程池变为TIDYING状态时，会执行钩子函数terminated()。terminated()在ThreadPoolExecutor类中是空的，若用户想在线程池变为TIDYING时，进行相应的处理；可以通过重载terminated()函数来实现。
      - 当线程池在SHUTDOWN状态下，阻塞队列为空并且线程池中执行的任务也为空时，就会由 SHUTDOWN -> TIDYING。 当线程池在STOP状态下，线程池中执行的任务为空时，就会由STOP -> TIDYING。
  - TERMINATED 
      - 线程池彻底终止，就变成TERMINATED状态。 
      - 线程池处在TIDYING状态时，执行完terminated()之后，就会由 TIDYING -> TERMINATED。
 
- Worker
    - Worker可以理解成线程池中任务执行的最小单元，实现了Runnable接口的run方法
    - Worker本身也实现了AQS，所以它也是一个锁，在执行任务的时候，会锁住自己，任务执行完成之后，会释放自己
 
- 如何设置线程池
    - IO 密集型任务：由于线程并不是一直在运行，所以可以尽可能的多配置线程，比如 CPU 个数 * 2
    - CPU 密集型任务（大量复杂的运算）应当分配较少的线程，比如 CPU 个数相当的大小。
    
- 优雅的关闭线程池
    - shutdown() 执行后停止接受新任务，会把队列的任务执行完毕。
    - shutdownNow() 也是停止接受新任务，但会中断所有的任务，将线程池状态变为 stop。
    
- 面试题
    - 队列再线程池中的作用
        - 当请求数大于coreSize时，可以让任务在队列上排队，让线程池中的线程慢慢的消费请求，工作中实际线程数不可能等于请求书，队列提供了一种机制让任务可以排队，缓冲区的作用
        - 当线程池消费完所有的线程后，会阻塞的从队列中拿数据，通过队列的阻塞功能是线程不消亡，一旦队列中有数据产生后，可立马被消费。
    - 线程池构造器参数的含义和表现
        - coreSize：核心线程数
        - maxSize: 最大线程数
        - keepAliveTime：线程空闲最大时间
        - queue：1、SynchronousQueue 为了避免任务被拒绝，要求线程池的maxSize无界，缺点是当任务提交的速度超过消费的速度时，可能出现无限制的线程增长；2、LinkedBlockingQueue 无界队列，未消费的任务可以在队列中等待； 3、ArrayBlockingQueue 有界队列，可以防止资源被耗尽
        - 在Executor已经关闭或对最大县城和最大队列都使用饱和时，可以使用RejectedExecutionHandler类进行捕获
    - 线程池中线程空闲回收的理解？
        - 空闲线程回收时机：如果线程超时，还从阻塞队列中拿不到任务，当前线程就会回收，如果`allowCoreThreadTimeOut`设置为true，core thread也会被回收，直到剩下一个线程为止，线程执行完成没有消亡，是因为阻塞的从队列中拿任务，在`keepAliveTime`的超时时间内还没拿到任务，就会打断阻塞，线程直接返回，线程的声明周期就结束了，JVM会回收掉该线程对象，所以回收线程的源码提现就是 让线程不在队列中阻塞，直接返回了。
    - 想在线程池任务执行前和执行后，做一些资源清理工作，如何操作？
        - ThreadPoolExecutor提供了一些钩子函数，我们只需要继承ThradPoolExecutor并实现这些钩子函数，在线程执行前实现beforeExecute方法，执行之后实现afterExecute方法。
    - 线程执行完任务后，做什么？
        - 阻塞从队列中拿任务，无任务就无限阻塞
        - 阻塞从队列中拿任务，没有任务阻塞一段时间，线程返回，被JVM回收
    - keepAliveTime设置成负数或者0，表示无限阻塞？
        - 这种是不对的，如果keepAliveTime设置成负数，线程池初始化就会直接抛异常；设置为0，队列如果是 LinkedBlockingQueue 的话， 执行 workQueue.poll (keepAliveTime, TimeUnit.NANOSECONDS) 方法时，如果队列中没 有任务，会直接返回 null，导致线程立马返回，不会无限阻塞。
        
- 经验
    - 初始化时设置coreSize == maxSize
        - 线程一下子增加到 maxSize，并且不要回收线程，防止线程回收， 避免不断增加回收的损耗。现在机器的资源都是很充足的，我们不用去担心线程空闲会浪费机器的资源
    - maxSize 无界 + SynchronousQueue
        - SynchronousQueue部有堆栈和队列两种形式，默认是 堆栈的形式，其内部是没有存储的容器的，放元素和拿元素是一一对应的，比如我使用 put 方 法放元素，如果此时没有对应的 take 操作的话，put 操作就会阻塞，需要有线程过来执行 take 操作后，put 操作才会返回。
        - 选择 SynchronousQueue 队列，假设所有请求都执行 put 操作，没有请求执行 take 操作，前 10 个 put 请求会消耗 10 个线程，都阻塞在 put 操作 上，第 11 个请求过来后，请求就会被拒绝，所以我们才说尽量把 maxSize 设置大一点，防止请求被拒绝。
    
        
        
        
        

  
    