- 使用
    - 下载zk包
    - bin目录下执行 `./zkServer.sh start`
    - 新启动端口连接 `./zkCli.sh`

- 配置文件
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
    
- 命令
    - `create /oitm`
    - `ls /`
    - `get /oitm`
    - `delete  /oitm/sub`

- 集群：根据配置文件中的配置来判断是否是集群
    ```
    clientPort=2181

    # 各个配置文件增加以下配置
    # 2887端口用来同步的，3887用来选举的
    server.1=localhost:2887:3887
    server.2=localhost:2888:3888
    server.3=localhost:2889:3889    
    ```
    - Leader、Follower、Observer
    - 必须要有一个Leader