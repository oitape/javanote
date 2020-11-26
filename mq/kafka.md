### 基本概念
- Broker : 和AMQP里协议的概念一样， 就是消息中间件所在的服务器
- Topic(主题) : 每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。(物理上不同Topic的消息 分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消 费数据而不必关心数据存于何处)
- Partition(分区) : Partition是物理上的概念，体现在磁盘上面，每个Topic包含一个或多个Partition.
- Producer : 负责发布消息到Kafka broker
- Consumer : 消息消费者，向Kafka broker读取消息的客户端。
- Consumer Group(消费者群组) : 每个Consumer属于一个特定的Consumer Group(可为每个Consumer 指定group name，若不指定group name则属于默认的group)。
- offset 偏移量: 是kafka用来确定消息是否被消费过的标识，在kafka内部体现就是一个递增的数字

### kafka特点
- 优点：基于磁盘的数据存储 高伸缩性 高性能。场景：收集信息、日志、流处理
- 缺点：运维难度大，偶尔有数据混乱情况，对zookeeper强依赖，多副本模式下对带宽有一定要求


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
- 列出消费群组：`./kafka-topics --new-consumer --bootstrap-server address:9092/kafka --list`
- 列出消费群组详细信息：`./kafka-topics --new-consumer --bootstrap-server address:9092/kafka --describe --group groupName`

### kafka生产者参数详解
- acks: 至少要多少个分区副本接收到了消息返回确认消息 一般是 0:只要消息发送出去了就确认(不管是否失败) 1:只 要 有一个broker接收到了消息 就返回 all: 所有集群副本都接收到了消息确认 当然 2 3 4 5 这种数字都可以， 就 是具体多少台机器接收到了消息返回， 但是一般这种情况很少用到
- buffer.memory: 生产者缓存在本地的消息大小 : 如果生产者在生产消息的速度过快 快过了往 broker发送消息 的速度 那么就会出现buffer.memory不足的问题 默认值为32M 注意 单位是byte 大概3355000左右
- max.block.ms: 生产者获取kafka元数据(集群数据，服务器数据等) 等待时间 : 当因网络原因导致客户端与服务器 通讯时等待的时间超过此值时 会抛出一个TimeOutExctption 默认值为 60000ms
- retries: 设置生产者生产消息失败后重试的次数 默认值 3次
- retry.backoff.ms: 设置生产者每次重试的间隔 默认 100ms
- batch.size: 生产者批次发送消息的大小 默认16k 注意单位还是byte
- linger.ms: 生产者生产消息后等待多少毫秒发送到broker 与batch.size 谁先到达就根据谁 默认值为0
- compression.type: kafka在压缩数据时使用的压缩算法 可选参数有:none、gzip、snappy 
    - none即不压缩 
    - gzip压缩率比较高 系统cpu占用比较大 但是带来的好处是 网络带宽占用少，     
    - snappy压缩比没有gzip高 cpu占用率不是很高 性能也还行， 如果网络带宽比较紧张的话 可以选择gzip 一般推荐 snappy
- client.id: 一个标识， 可以用来标识消息来自哪， 不影响kafka消息生产
- max.in.flight.requests.per.connection: 指定kafka一次发送请求在得到服务器回应之前,可发送的消息数量

### kafka消费者参数
- fetch.min.bytes:<br>
该属性指定了消费者从服务器获取记录的最小字节数。broker 在收到消费者的数据请求时，如果可用的数据量小 于 fetch.min.bytes 指定的大小，那么它会等到有足够的可用数据时才把它返回给消费者。这样可以降低消费者和 broker 的工作负载，因为它们在主题不是很活跃的时候(或者一天里的低谷时段)就不需要来来回回地处理消 息。如果没有很多可用数据，但消费者的 CPU 使用率却很高，那么就需要把该属性的值设得比默认值大。如果消 费者的数量比较多，把该属性的值设置得大一点可以降低 broker 的工作负载。 默认值为1 byte
- fetch.max.wait.ms <br>用于指 定 broker 的等待时间，默认是如果没有足够的数据流入Kafka，消费者获取最小数据量的要求就得不到满足，最终 导致 500ms 的延迟。如果 fetch.max.wait.ms 被设为 100ms，并且 fetch.min.bytes 被设为 1MB，那么 Kafka 在 收到消费者的请求后，要么返回 1MB 数据，要么在 100ms 后返回所有可用的数据，就看哪个条件先得到满足。 默认值为500ms
- max.partition.fetch.bytes 该属性指定了服务器从每个分区里返回给消费者的最大字节数。默认值是 1MB
- session.timeout.ms 和heartbeat.interval.ms
    - session.timeout.ms : 消费者多久没有发送心跳给服务器，服务器则认为消费者死亡/退出消费者组 默认值:10000ms 
    - heartbeat.interval.ms :消费者往kafka服务器发送心跳的间隔 一般设置为session.timeout.ms的三分之一 默认值:3000ms
- auto.offset.reset: 当消费者本地没有对应分区的offset时 会根据此参数做不同的处理 默认值为:latest
    - earliest：当各分区下有已提交的offset时，从提交的offset开始消费;无提交的offset时，从头开始消费
    - latest 当各分区下有已提交的offset时，从提交的offset开始消费;无提交的offset时，消费新产生的该分区下的数据
    - none topic各分区都存在已提交的offset时，从offset后开始消费;只要有一个分区不存在已提交的offset，则抛出异常
enable.auto.commit
该属性指定了消费者是否自动提交偏移量，默认值是 true。为了尽量避免出现重复数据和数据丢失，可以把它设为 false，由自己控制何时提交偏移量。如果把它设为 true，还可以通过配置 auto.commit.interval.ms 属性来控制 提交的频率。
partition.assignment.strategy
PartitionAssignor 根据给定的消费者和主题，决定哪些分区应该被分配给哪个消费者。Kafka 有两个默认的分配策
略。Range:该策略会把主题的若干个连续的分区分配给消费者。假设消费者 C1 和消费者 C2 同时订阅了主题 T1 和主题 T2，并且每个主题有 3 个分区。那么消费者 C1 有可能分配到这两个主题的分区 0 和分区 1，而消费 者 C2 分配到这两个主题的分区2。因为每个主题拥有奇数个分区，而分配是在主题内独立完成的，第一个消 费者最后分配到比第二个消费者更多的分区。只要使用了 Range 策略，而且分区数量无法被消费者数量整 除，就会出现这种情况。
RoundRobin:该策略把主题的所有分区逐个分配给消费者。如果使用 RoundRobin 策略来给消费者 C1 和消 费者 C2 分配分区，那么消费者 C1 将分到主题 T1 的分区 0 和分区 2 以及主题 T2 的分区 1，消费者 C2 将分 配到主题 T1 的分区 1 以及主题 T2 的分区 0 和分区 2。一般来说，如果所有消费者都订阅相同的主题(这种 情况很常见)，RoundRobin 策略会给所有消费者分配相同数量的分区(或最多就差一个分区)。
max.poll.records
单次调用 poll() 方法最多能够返回的记录条数 ,默认值 500
receive.buffer.bytes 和 send.buffer.bytes
receive.buffer.bytes 默认值 64k 单位 bytes
send.buffer.bytes 默认值 128k 单位 bytes
这两个参数分别指定了 TCP socket 接收和发送数据包的缓冲区大小。如果它们被设为 -1

### 生产者和消费者  与分区关系
- 生产者
    - 如果发送消息的时候制定了分区，则消息投递到指定的分区
    - 如果没有指定分区，且消息的key不为空，则基于key的哈希值来选择一个分区
    - 如果既没有指定分区，消息的key也是空，则用轮训的方式选择一个分区
- 消费者 
 消费者以组的名义订阅主题，主题有多个分区，消费者组中有多个消费者实例，同一时刻，一条消息只能被组中的一个消费者实例消费
 - 主题下的每个分区只从属于组中的一个消费者，不可能出现组中的两个消费者负责同一个分区
 - 如果分区数大于或者等于组中的消费者实例数，无非一个消费者会负责多个分区；如果消费者实例的数量大于分区数，那么按照默认的策略(可自定义策略)，有一些消费者是多余的，一直接不到消息而处于空闲状态。
 - Kafka它在设计的时候就是要保证分区下消息的顺序，也就是说消息在一个分区中的顺序是怎样的，那么消费者在消费的时候看到的就是什么样的顺序，那么要做到这一点就首先要保证消息是由消费者主动拉取的（pull），其次还要保证一个分区只能由一个消费者负责。
 - 可以指定消费哪些分区消费

```java
 List<TopicPartition> list = new ArrayList<>(); 
 //new出一个分区对象 声明这个分区是哪个topic下面的哪个分区 
 list.add(new TopicPartition("test-topic",0)); 
 //分配这个消费者所需要消费的分区, 传入一个分区对象集合      
 kafkaConsumer.assign(list);
```

### 自动提交的问题
    - 当A本地消费的偏移量还没有提交时，topic消费组中新加入了消费者B，发生了分区再均衡，如果A消费到offset 6，没有提交，服务器中分区的offset 是4，然后该分区均衡给了B，那么B就从offset 4进行消费，发生了消息重复消费。
    - 消费者可以监听kafka的分区再均衡，分别有两个回调方法
        - 分区再均衡之前方法（同步提交消费者本地offset）
        - 分区再均衡之后方法





