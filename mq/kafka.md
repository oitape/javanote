### 基本概念
- Broker : 和AMQP里协议的概念一样， 就是消息中间件所在的服务器
- Topic(主题) : 每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。(物理上不同Topic的消息 分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消 费数据而不必关心数据存于何处)
- Partition(分区) : Partition是物理上的概念，体现在磁盘上面，每个Topic包含一个或多个Partition.
- Producer : 负责发布消息到Kafka broker
- Consumer : 消息消费者，向Kafka broker读取消息的客户端。
- Consumer Group(消费者群组) : 每个Consumer属于一个特定的Consumer Group(可为每个Consumer 指定group name，若不指定group name则属于默认的group)。
- offset 偏移量: 是kafka用来确定消息是否被消费过的标识，在kafka内部体现就是一个递增的数字

### server参数
- log.dirs: 日志文件存储地址， 可以设置多个
- num.recovery.threads.per.data.dir:用来读取日志文件的线程数量，对应每一个log.dirs 若此参数为2 log.dirs 为 2个目录 那么就会有4个线程来读取
- auto.create.topics.enable:是否自动创建tiopic
- num.partitions: 创建topic的时候自动创建多少个分区 (可以在创建topic的时候手动指定) 
- log.retention.hours: 日志文件保留时间 超时即删除
- log.retention.bytes: 日志文件最大大小
- log.segment.bytes: 当日志文件达到一定大小时，开辟新的文件来存储(分片存储) 
- log.segment.ms: 同上 只是当达到一定时间时 开辟新的文件
- message.max.bytes: broker能接收的最大消息大小(单条) 默认1M

### kafka基本管理操作命令
- 列出所有主题：`./kafka-topics --zookeeper zkaddress:2181/kafka --list`
- 列出所有主题详细信息：`./kafka-topics --zookeeper zkaddress:2181/kafka --describe`
- 创建主题指定副本分区：`./kafka-topics --zookeeper zkaddress:2181/kafka --create --replication-factor 1 --partitions 8 --topic topicName`
- 增加分区，分区无法被删除：`./kafka-topics --zookeeper zkaddress:2181/kafka  --alter --topic tppicName --partitions 3`
- 删除主题：`./kafka-topics --zookeeper zkaddress:2181/kafka --delete --topic topicName`
- 列出消费群组：``
- 列出消费群组详细信息：`./kafka-topics --new-consumer --bootstrap-server address:9092/kafka --list`
