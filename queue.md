- LinkedBlockingQueue:链表阻塞队列
    ![](/assets/iShot2020-09-15下午04.53.45.png)
    - queue是最基础的接口，几乎所有的队列实现类都会实现这个接口，该接口有三大类操作：
        - 新增操作
            - add队列满了抛出异常
            - offer队列满了返回false
        - 查看并删除
            - remove队列空的时候抛异常
            - poll队列空的时候返回null
        - 查看不删除
            - element队列空的时候抛异常
            - peek队列空的时候返回null
        - 
    