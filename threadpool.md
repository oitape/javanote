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
    - newCachedThreadPool
        ```java
        new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
        ```
    - newSingleThreadExecutor
        ```java
        new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
        ```
    - newScheduledThreadPool
        ```java
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue());
        ```
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
  
  
  
    