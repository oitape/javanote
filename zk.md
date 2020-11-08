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

    * 共同配置

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

    * zoo1.cfg
      ```
      dataDir=/Users/Oitm/Downloads/Software/apache-zookeeper-3.6.2-bin/zookeeper/data_1
      dataLogDir=/Users/Oitm/Downloads/Software/apache-zookeeper-3.6.2-bin/zookeeper/logs_1
      clientPort=2181
      ```
    * zoo2.cfg
      ```
      dataDir=/Users/Oitm/Downloads/Software/apache-zookeeper-3.6.2-bin/zookeeper/data_2
      dataLogDir=/Users/Oitm/Downloads/Software/apache-zookeeper-3.6.2-bin/zookeeper/logs_2
      clientPort=2182
      ```
    * zoo3.cfg
      ```
      dataDir=/Users/Oitm/Downloads/Software/apache-zookeeper-3.6.2-bin/zookeeper/data_3
      dataLogDir=/Users/Oitm/Downloads/Software/apache-zookeeper-3.6.2-bin/zookeeper/logs_3
      clientPort=2183
      ```
    * zoo\_ob.cfg
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

* 为什么要有observer

  * Zookeeper 服务中的每个 Server 可服务于多个 Client，并且 Client 可连接到 ZK 服务中的任一台 Server？来提交请求。若是读请求，则由每台 Servere 的本地副本数据库直接响应。若是改变 Server 状态的写请求，需要通过一致性协议来处理，这个协议就是 Zab 协议。

  * Zab 协议规定：来自 Client 的所有写请求，都要转发给 ZK 服务中唯一的 Server-- Leader，由 Leader 根据该请求发起一个 Proposal。然后，其他的 Server 对该 Proposal 进行投票。之后，Leader 对投票进行收集当投票数量过半时 Leader会向所有的 server/发送一个通知消息。最后，当 Clienta 所连接的 Server 收到该消息时，会把该操作更新到内存中并对 Cliente 的写请求做出回应

  * Zookeeper 服务器在上述协议中实际扮演了两个职能。它们一方面从客户端接受连接与操作请求，另一方面对操作结果进行投票。这两个职能在 ZooKeeper 集群扩展的时候彼此制约。例如，当我们希望增加 ZK 服务中  Client？数量的时候，那么我们就需要增加 Servers 的数量，来支持这么多的客户端。然而，从 Zab 协议对写请求的处理过程中我们可以发现，增加服务器的数量，则增加了对协议中投票过程的压力。因为 Leader 节点必须等待集群中过半 Serverl 应投票，于是节点的增加使得部分计算机运行较慢，从而拖慢整个投票过程的可能性也随之提高，写操作也会随之下降。这正是我们在实际操作中看到的向题一一随着 Zookeeper 集群变大，写操作的吞吐量会下降。所以，我们不得不，在增加 Client 数量的期望和我们希望保持较好吞吐性能的期望间进行权衡。要打破这一耦合关系，我们引入了不参与投票的服务器，称为 Observer。Observer 可以接受客户端的连接，并将写请求转发给 Leader-节点。但是，Leader 节点不会要求 Observer 参加投票。相反，Observer。不参与投票过程，仅仅在上述第 3 歩那样，和其他服务节点一起得到投票结果。
  * 这个简单的扩展，给 Zookeeper 的可伸缩性帯来了全新的镜像。我们现在可以加入很多 Observer 节点，而无须担心严重影响写吞吐量。但他并非是无懈可击的，因为协议中的通知阶段，仍然与服务器的数量呈线性关系。但是，这里的串行开销非常低。因此，我们可以认为在通知服务器阶段的开销无法成为主要瓶颈。

* ZK中的CAP

  * Zookeepers 至少满足了 CP，牺牲了可用性，比如现在集群中有 Leader和 Follower 两种角色，那么当其中任意台服务器挂掉了，都要重新进行选举，在选举过程中，集群是不可用的，这就是牺牲的可用性
  * 但是，如果集群中有 Leader、Follower、Observer 三种角色，那么如果挂掉的是 Observer，那么对于集群来说并没有影响，集群还是可以用的，只是 Observer 节点的数据不同了，从这个角度考虑，Zookeeper）又是牺牲了一致性，满足了AP。

* znode节点类型

  * 持久节点：节点创建后就一直存在，直到有删除操作来主动清除这个节点`create /node "content"`  节点必须以 `/` 开始，创建已存在的节点会报错
  * 临时节点：临时节点的生命周期和客户端会话绑定，如果客户端会话失效，那么这个节点就会自动清除，会话失效而非连接断开。此外临时节点下面不能创建子节点。`create -e /node-e "content"`
  * 持久顺序节点：这类节点的基本特性和持久节点是一致的。额外的特性是，在 ZK 中，每个父节点会为他的第一级子节点维护一份时序，会记录每个子节点创建的先后顺序。基于这个特性，在创建子节点的时候，可以设置这个属性，那么在创建节点过程中，ZK 会自动为给定节点名加上ー个数字后缀，作为新的节点名。这个数字后缀的范围是整型的最大值 `create -s /node-s/prefix_  "content"`  创建出的节点`/node-s/prefix_0000000000`
  * 临时顺序节点：类似临时节点和顺序节点

* zk默认对每个节点最大的数据量是 **0xfffff**，1M字节，可配置修改

* stat：使用命令 ** ls -s /node **  或者 **stat /node**

  * Zookeeper命名空间中的每一个znode都有一个与之关联的stat结构，类似于Linux中文件的stat结构，znode的stat结构中字段显示如下：
    * cZxid：创建znode的事务ID
    * mZxid：最后修改znode的事务ID
    * pZxid：最后修改或删除子节点的事务ID
    * ctime：表示从 1970-01-01100:00:00Z 开始以毫秒为单位的 anode 创建时间
    * mtime
    * dataVersion：表示对该znode的数据所做的更改次数
    * cversion：表示对此znode的子节点进行更改的次数
    * aclVersion：表示对此znode的ACL进行更改的次数
    * ephemeralowner：如果 znode 是 ephemeral 类型节点，则这是 znode所有者的 session D。如果 znode不是 ephemeral 节点，则该字段设置为零
    * dataLength: 这是znode数据字段的长度
    * numChildren：这表示znode的子节点的数量

* Zxid

  * 类似于RDBMS中的事务ID，用于标识一次更新操作的Proposal ID。为了保证顺序性，改zxid必须单调递增。因为zk使用的是64位的数表示，高32位是Leader的epoch，从1开始，每次选出新的Leader，epoch加一。低32位为该epoch内的序号，每次epoch变化，都将低32位的序列号重置，这样保证zkid的全局递增性

* watch

  * 一个zk的节点可以被监控，包括这个目录中存储的数据的修改，子节点目录的变化，一旦变化可以通知设置监控的客户端，这个功能是zookeeper对于一个应用最重要的特性，通过这个特性可以实现的功能：配置集中管理、集群管理、分布式锁等等。
  * watch机制：一个watch事件是一个一次性的触发器，当贝设置watch的数据发生改变的时候，则服务器将这个改变发送给设置watch的客户端
  * 可以注册watcher的方法：getData、exists、getChildren
  * 可以触发watcher的方法：create、delete、setData。连接断开的情况下触发的watcher会丢失。
  * 一个watcher实例是一个回调函数，被回调一次就被移除了。如果还需要继续关注数据的变化，需要再次注册watcher。

* ACL权限控制： `schema:id:permission`

  * schema: 鉴权策略
    * world：只有一个用户：anyone，代表所有人认）
    * ip：使用 IP 地址认证
    * auth：使用已添加认证的用户认证  
    * digest：使用用户名：密码方式认证
  * id: 授权对象 ID 是指，权限赋予的用户或者一个实体
    * world：只有个 id，即 anyone
    * ip: 通常是一个p地址或地址段,比如192.168.0.110或192.168.0.1/24     
    * auth：用户名
    * digest：自定义：通常是“username: BASE64 \(SHA-1 \(username password\)
  * 权限
    * CREATE，简写为 c，可以创建子节点  
    * DELETE，简写为 d，可以初除子节点仅下一级节点），注意不是本节点
    * READ，简写为 r，可以读取节点数据及显示子节点列表
    * WRITE，简写为 w，可设置节点数据
    * ADMN，简写为 a，可以设置节点访问控制列表

* 命令

  * create  /node/subnode
  * delete  /oitm
  * deleteall  /node/subnode
  * set  /node  data
  * get  /node
  * stat  /node
  * getAcl  /node
  * setAcl  /node world:anyone:cdrwa

* zk常用客户端使用

  * Curator
    * 依赖
      ```xml
      <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-recipes</artifactId>
        <version>5.1.0</version>
      </dependency>
      <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-framework</artifactId>
        <version>5.1.0</version>
      </dependency>
      <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-client</artifactId>
        <version>5.1.0</version>
      </dependency>
      ```
    * 使用
      ```java
      CuratorFramework curatorFramework = CuratorFrameworkFactory.newClient("localhost:2181", new RetryNTimes(3, 1000));
      curatorFramework.start();
      //curatorFramework.create().forPath("/data", "content".getBytes());
      CuratorCache curatorCache = CuratorCache.builder(curatorFramework, "/data").build();
      curatorCache.start();
      curatorCache.listenable().addListener(new CuratorCacheListener() {
          @Override
          public void event(Type type, ChildData oldData, ChildData data) {
              System.out.println(type);
              System.out.println(oldData);
              System.out.println(data);
          }
      });
      System.in.read();
      ```
- ZK如何保证数据一致性
  - FollowerRequestProcessor为Follower的首个处理器，如果是写请求，先将请求写入commitprocessor的queuedRequests（方便后续commit时判断是否本Follower提交的写请求），然后转Leader
  - Leader为每个请求生成zxid，下发proposal给Follower，Follower会将请求写入到pendingTxns阻塞队列及txnLog中，然后发送ack给Leader（proposal这步是会发给所有的follower的（放到LearnerHandler的请求处理队列中，一个Follower一个LearnerHandler），之后Follower的ack就不一定全返回了）
  - ack过半，Leader发送commit请求给所有Follower，Follower对比commit request的zxid和前面提到的pendingTxns的zxid，不一致的话Follower退出，重新跟Leader同步
  - Follower处理commit请求，如果不是本Follower提交的写请求，直接调用FinalRequestProcessor做持久化，触发watches；如果是本Follower提交，则做一些特殊处理（主要针对客户端连接断开的场景），然后调用FinalRequestProcessor等后续处理流程
  - FinalRequestProcessor做持久化，返回客户端
  - 对于zookeeper来说，它实现了A可用性、P分区容错性、C中的写入强一致性，丧失的是C中的读取一致性
  
  
- Leader选举：是保证分布式数据一致性的关键所在。
  - 当ZK集群中的一台服务器出现以下情况就会进行Leader选举
    - 服务器初始化成功
    - Leader挂掉
  - 服务器启动时的Leader选举
    - 若进行 Leader 选举，则至少需要两台机器，这里选取 3 台机器组成的服务器集群为例。在集群初始化阶段，当有一台服务器 Server1 启动时，其单独无法进行和完成 Leaderi 选举，当第二台服务器 Server2 启动时，此时两台机器可以相互通信，每台机器都试图找到 Leader，于是进入 Leaderi 选举过程。选举过程如下
        - 每个 Server？发出一个投票。由于是初始情况，Server 和 Server2 都会将自己作为 Leader 服务器来进行投票，每次投票会包含所推举的服务器的 myid 和 ZXID，使用（myid, ZXID）来表示，此时 Server1 的投票为（1,), Server2 的投票为（2,0），然后各自将这个投票发给集群中其他机器 
        - 接受来自各个服务器的投票。集群的每个服务器收到投票后，首先判断该投票的有效性，如检查是否是本轮投票（检查 ZXID）、是否来自 OOKING 状态的服务器
       - 处理投票。针对每一个投票，服务器都需要将人的投票和自己的投票进行 PK, PK 规则如下
            - 优先检查 ZXD。ZXID 比较大的服务器优先作为 Leader
            - 如果 ZXID 相同，那么就比较 myid。myid 较大的服务器作为 Leader 服务器
            - 对于 Server1 而言，它的投票是（1,0），接收 Server2 的投票为（2,0），首先会比较两者的 ZXID，均为 0, 再比较 myid，此时 Server2 的 myid 最大，于是更新自己的投票为（2,0），然后重新投票，对于 Server2 而言，其无须更新自己的投票，只是再次向集群中所有机器发出上一次投票信息即可
       - 统计投票。每次投票后，服务器都会统计投票信息，判断是否已经有过半机器接受到相同的投票信息对于 Server1、Server2 而言，都统计出集群中已经有两台机器接受了（2，）的投票信息，此时便认为已经选出了 Leader。
      - 改变服务器状态确定了 Leader，每个服务器就会更新自己的状态，如果是 Follower，那么就变更为 FOLLOWING，如果是 Leader，就变更为 LEADING

