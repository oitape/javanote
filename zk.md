- 使用
    - 下载zk包
    - bin目录下执行 `./zkServer.sh start`
    - 新启动端口连接 `./zkCli.sh`

- 配置文件
    ```
    # 基本单位  2000ms
    tickTime=2000
    # 10个tickTime
    initLimit=10
    # 5个tickTime
    syncLimit=5
    # 快照数据  事务日志 两者可分开配置
    dataDir=/tmp/zookeeper
    # 客户端连接端口
    clientPort=2181
    ```
    
- 命令
    - `create /oitm`
    - `ls /`
    - `get /oitm`
    - `delete  /oitm/sub`

- 集群：根据配置文件中的配置来判断是否是集群
    ```
    clientPort=2181

    server.1=localhost:2887:3887
    server.2=localhost:2888:3888
    server.3=localhost:2889:3889    
    ```
    - Leader、Follower、Observer
    - 必须要有一个Leader