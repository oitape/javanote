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
 
- 如何设置线程池
    - IO 密集型任务：由于线程并不是一直在运行，所以可以尽可能的多配置线程，比如 CPU 个数 * 2
    - CPU 密集型任务（大量复杂的运算）应当分配较少的线程，比如 CPU 个数相当的大小。
    
- 优雅的关闭线程池
    - shutdown() 执行后停止接受新任务，会把队列的任务执行完毕。
    - shutdownNow() 也是停止接受新任务，但会中断所有的任务，将线程池状态变为 stop。
    


  
    