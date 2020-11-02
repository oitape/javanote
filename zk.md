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

- 集群
    ```
    
    ```