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
    - zoo1.cfg
    
    ```
    # 基本单位  2000ms
    tickTime=2000
    # 10个tickTime
    initLimit=10
    # 5个tickTime
    syncLimit=5
    # 快照数据  事务日志 两者可分开配置
    dataDir=/Users/Oitm/Downloads/Software/apache-zookeeper-3.6.2-bin/zookeeper/data_1
    dataLogDir=/Users/Oitm/Downloads/Software/apache-zookeeper-3.6.2-bin/zookeeper/logs_1
    # 客户端连接端口
    clientPort=2181
    
    server.1=localhost:2887:3887
    server.2=localhost:2888:3888
    server.3=localhost:2889:3889
    server.4=localhost:2890:3890:observer    
    ```
    - zoo2.cfg
    ```
    
    ```    
    

  * 创建数据以及日志文件目录

    ```sh
    mkdir -p /zookeeper/data_1
    mkdir -p /zookeeper/data_2
    mkdir -p /zookeeper/data_3

    mkdir -p /zookeeper/logs_1
    mkdir -p /zookeeper/logs_2
    mkdir -p /zookeeper/logs_3
    ```

  * 创建myid文件

    ```
    echo "1" > /zookeeper/data_1/myid
    echo "2" > /zookeeper/data_2/myid
    echo "3" > /zookeeper/data_3/myid
    ```

  * 修改配置文件

    ```
    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/zookeeper/data_1
    clientPort=2181
    dataLogDir=/zookeeper/logs_1
    # server.x􏱱􏱋中x和􏰻myid中保持一致  􏱱
    server.1=localhost:2887:3887 
    server.2=localhost:2888:3888         
    server.3=localhost:2889:3889
    ```

  * 启动集群

    ```
    zkServer.sh start {dir}/conf/zoo1.cfg
    zkServer.sh start {dir}/conf/zoo2.cfg
    zkServer.sh start {dir}/conf/zoo3.cfg
    ```

  * 验证

    ```
    zkServer.sh status {dir}/conf/zoo1.cfg
    zkServer.sh status {dir}/conf/zoo2.cfg
    zkServer.sh status {dir}/conf/zoo3.cfg
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



