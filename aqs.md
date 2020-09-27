- AQS：AbstractQueuedSychronizer同步发生器，构建LOCK。在JUC包ReentrantLock、读写锁、信号量的基础都是AQS
  - 通过内置得到FIFO同步队列来完成线程争夺资源的管理工作
  - 同步器来管理 需要获取资源的Node线程队列，每个线程做的事情就是获取锁、释放锁，然后队列里的线程通过自旋锁去公平竞争资源

