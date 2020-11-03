* 使用

  * 下载zk包
  * bin目录下执行 `./zkServer.sh start`
  * 新启动端口连接 `./zkCli.sh`

* 配置文件

  ```
    # 服务器之间或者客户端与服务器之间心跳时间  2000ms，zookeeper里也有类似session的概念，zookeeper里最小的session过期时间是tickTime的两倍
    tickTime=2000
    # 10个tickTime，Follower在启动过程中，会从Leader同步所有最新数据，然后确定自己能够对外服务的起始状态，Leader允许F在initLimit时间内完成这个工作，如果集群数据量大，Follower在启动的时候，从Leader上同步数据的时间也会变长，需要适当调大这个参数
    initLimit=10
    # 5个tickTime ，Leader负责与zk集群中所有机器通信，如果Leader发出心跳包在syncLimit之后还没收到Follower哪里收到响应，那么就认为这个Follower已经不在线
    syncLimit=5
    # 快照数据  默认情况下事务日志也在这里  两者可分开配置，通过配置dataLogDir参数记录事务日志，事务的写性能直接影响zk性能
    dataDir=/tmp/zookeeper
    # 客户端连接端口
    clientPort=2181

    maxClientCnxns
    autopurge.snapRetainCount
    autopurge.purgeInterval
  ```

* 命令

  * `create /oitm`
  * `ls /`
  * `get /oitm`
  * `delete  /oitm/sub`

* 集群：根据配置文件中的配置来判断是否是集群；Leader、Follower、Observer

  * 配置文件
    - 共同配置
    
    ```
    # 基本单位  2000ms
    tickTime=2000
    # 10个tickTime
    initLimit=10
    # 5个tickTime
    syncLimit=5  
    
    server.1=localhost:2887:3887
    server.2=localhost:2888:3888
    server.3=localhost:2889:3889
    server.4=localhost:2890:3890:observer  
    ```
    
    - zoo1.cfg
    ```
    dataDir=/Users/Oitm/Downloads/Software/apache-zookeeper-3.6.2-bin/zookeeper/data_1
    dataLogDir=/Users/Oitm/Downloads/Software/apache-zookeeper-3.6.2-bin/zookeeper/logs_1
    clientPort=2181
    ```
    - zoo2.cfg
    ```
    dataDir=/Users/Oitm/Downloads/Software/apache-zookeeper-3.6.2-bin/zookeeper/data_2
    dataLogDir=/Users/Oitm/Downloads/Software/apache-zookeeper-3.6.2-bin/zookeeper/logs_2
    clientPort=2182
    ```    
    - zoo3.cfg
    ```
    dataDir=/Users/Oitm/Downloads/Software/apache-zookeeper-3.6.2-bin/zookeeper/data_3
    dataLogDir=/Users/Oitm/Downloads/Software/apache-zookeeper-3.6.2-bin/zookeeper/logs_3
    clientPort=2183
    ```
    - zoo_ob.cfg
    ```
    dataDir=/Users/Oitm/Downloads/Software/apache-zookeeper-3.6.2-bin/zookeeper/data_4
    dataLogDir=/Users/Oitm/Downloads/Software/apache-zookeeper-3.6.2-bin/zookeeper/logs_4
    clientPort=2184
    peerType=observer
    ```
    

    

  * 创建数据以及日志文件目录

    ```sh
    mkdir -p {path}/zookeeper/data_1
    mkdir -p {path}/zookeeper/data_2
    mkdir -p {path}/zookeeper/data_3
    mkdir -p {path}/zookeeper/data_4

    mkdir -p {path}/zookeeper/logs_1
    mkdir -p {path}/zookeeper/logs_2
    mkdir -p {path}/zookeeper/logs_3
    mkdir -p {path}/zookeeper/logs_4
    ```

  * 创建myid文件

    ```
    echo "1" > {path}/zookeeper/data_1/myid
    echo "2" > {path}/zookeeper/data_2/myid
    echo "3" > {path}/zookeeper/data_3/myid
    echo "4" > {path}/zookeeper/data_3/myid
    ```

  * 启动集群

    ```
    zkServer.sh start {dir}/conf/zoo1.cfg
    zkServer.sh start {dir}/conf/zoo2.cfg
    zkServer.sh start {dir}/conf/zoo3.cfg
    zkServer.sh start {dir}/conf/zoo_ob.cfg
    ```

  * 验证

    ```
    zkServer.sh status {dir}/conf/zoo1.cfg
    zkServer.sh status {dir}/conf/zoo2.cfg
    zkServer.sh status {dir}/conf/zoo3.cfg
    zkServer.sh status {dir}/conf/zoo_ob.cfg
    ```

  * 使用三个客户端分别连接三台服务器

    ```
    zkCli.sh -server localhost:2181
    zkCli.sh -server localhost:2182
    zkCli.sh -server localhost:2183
    ```

* Observer

  * 配置修改: 
    * 节点配置增加`peerType=observer`
    * servers列表增加`server.4=localhost:2890:3890:observer`

* 集群角色
  * Leader：负责进行投票的发起和决议，最终更新状态
  * Follower：用于接收客户端请求并返回客户结果。参与Leader发起的投票
  * observer：可以接收客户端连接，将写请求转发给leader节点。但是Observer不参与投票过程，只是同步leader的状态。

- 为什么要有observer
  - Zookeeper 服务中的每个 Server 可服务于多个 Client，并且 Client 可连接到 ZK 服务中的任一台 Server？来提交请求。若是读请求，则由每台 Servere 的本地副本数据库直接响应。若是改变 Server 状态的写请求，需要通过一致性协议来处理，这个协议就是 Zab 协议。

  - Zab 协议规定：来自 Client 的所有写请求，都要转发给 ZK 服务中唯一的 Server-- Leader，由 Leader 根据该请求发起一个 Proposal。然后，其他的 Server 对该 Proposal 进行投票。之后，Leader 对投票进行收集当投票数量过半时 Leader会向所有的 server/发送一个通知消息。最后，当 Clienta 所连接的 Server 收到该消息时，会把该操作更新到内存中并对 Cliente 的写请求做出回应
  - Zookeeper 服务器在上述协议中实际扮演了两个职能。它们一方面从客户端接受连接与操作请求，另一方面对操作结果进行投票。这两个职能在 ZooKeeper 集群扩展的时候彼此制约。例如，当我们希望增加 ZK 服务中  Client？数量的时候，那么我们就需要增加 Servers 的数量，来支持这么多的客户端。然而，从 Zab 协议对写请求的处理过程中我们可以发现，增加服务器的数量，则增加了对协议中投票过程的压力。因为 Leader 节点必须等待集群中过半 Serverl 应投票，于是节点的增加使得部分计算机运行较慢，从而拖慢整个投票过程的可能性也随之提高，写操作也会随之下降。这正是我们在实际操作中看到的向题一一随着 Zookeeper 集群变大，写操作的吞吐量会下降。所以，我们不得不，在增加 Client 数量的期望和我们希望保持较好吞吐性能的期望间进行权衡。要打破这一耦合关系，我们引入了不参与投票的服务器，称为 Observer。Observer 可以接受客户端的连接，并将写请求转发给 Leader-节点。但是，Leader 节点不会要求 Observer 参加投票。相反，Observer。不参与投票过程，仅仅在上述第 3 歩那样，和其他服务节点一起得到投票结果。
  - 这个简单的扩展，给 Zookeeper 的可伸缩性帯来了全新的镜像。我们现在可以加入很多 Observer 节点，而无须担心严重影响写吞吐量。但他并非是无懈可击的，因为协议中的通知阶段，仍然与服务器的数量呈线性关系。但是，这里的串行开销非常低。因此，我们可以认为在通知服务器阶段的开销无法成为主要瓶颈。

- ZK中的CAP
  - Zookeepers 至少满足了 CP，牺牲了可用性，比如现在集群中有 Leader？和 Follower 两种角色，那么当其中任意台服务器挂掉了，都要重新进行选举，在选举过程中，集群是不可用的，这就是牺牲的可用性
  - 但是，如果集群中有 Leader、Follower、Observer 三种角色，那么如果挂掉的是 Observer，那么对于集群来说并没有影响，集群还是可以用的，只是 Observer 节点的数据不同了，从这个角度考虑，Zookeeper）又是牺牲了一致性，满足了AP。
  
