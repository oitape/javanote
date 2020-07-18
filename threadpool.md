- 线程池
    - newFixedThreadPool
        ```java
        new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
        ```
    - newCachedThreadPool
        ```java
        
        ```