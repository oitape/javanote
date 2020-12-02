# redis

## redis单机安装
1.官网下载redis包解压
2.yum install gcc-c++   ||  apt install gcc
3.make
4.make install
redis默认安装位置 /usr/local/bin

## redis配置外网可访问的几种方式

```
 1.注释 bind && 设置protected-mode no
 2.使用bind 0.0.0.0 && 没有设置密码
 3.设置密码
 protected-mode它启用的条件有两个：第一是没有使用bind，第二是没有设置访问密码。
```

## redis数据类型及api操作(http://redisdoc.com/)

#### key

keys *  

scan 0 match  *  count  1

exists key 判断某个key是否存在

move key db  当前库就没有了，到指定的库中去了

expire key  为给定的key设置过期时间

ttl key 查看还有多少时间过期   -1表示永不过期  -2表示已过期

type key  查看key是什么类型

#### 1.string

string是redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value。

string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象 。

string类型是Redis最基本的数据类型，一个redis中字符串value最多可以是512M

set  key  value   设置key  value

get  key    查看当前key的值

del  key   删除key

append key  value   如果key存在，则在指定的key末尾添加，如果key存在则类似set

strlen  key  返回此key的长度

以下几个命令只有在key值为数字的时候才能正常操作

------

incr  key  为执定key的值加一

decr  key  为指定key的值减一

incrby key  数值     为指定key的值增加数值

decrby key  数值     为指定key的值减数值

------

getrange  key  0(开始位置)  -1（结束位置）    获取字符串指定区间范围内的子串

setrange key 1（开始位置，从哪里开始设置） 具体值    设置（替换）指定区间范围内的值

setex 键 秒值 真实值    设置带过期时间的key，动态设置。

setnx  key   value         只有在 key 不存在时设置 key 的值。

mset   key1   value  key2   value       同时设置一个或多个 key-value 对。

mget   key1   key 2    获取所有(一个或多个)给定 key 的值。

msetnx   key1   value  key2   value   同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。

getset   key    value  将给定 key 的值设为 value ，并返回 key 的旧值(old value)。



#### 2.list

它是一个字符串链表，left、right都可以插入添加；
如果键不存在，创建新的链表；
如果键已存在，新增内容；
如果值全移除，对应的键也就消失了。
链表的操作无论是头和尾效率都极高，但假如是对中间元素进行操作，效率就很惨淡了。

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边）。
它的底层实际是个链表

lpush  key  value1  value2  将一个或多个值加入到列表头部

rpush  key  value1  value2 将一个或多个值加入到列表底部

lrange key  start  end  获取列表指定范围的元素   （0 -1）表示全部

lpop key 移出并获取列表第一个元素

rpop key  移出并获取列表最后一个元素

lindex key index   通过索引获取列表中的元素 

llen 获取列表长度

 lrem key   0（数量） 值，表示删除全部给定的值。零个就是全部值   从left往right删除指定数量个值等于指定值的元素，返回的值为实际删除的数量

ltrim key  start(从哪里开始截)  end（结束位置） 截取指定索引区间的元素，格式是ltrim list的key 起始索引 结束索引



#### 3.set

Redis的Set是string类型的无序，不能重复的集合。

sadd key value1 value 2 向集合中添加一个或多个成员

smembers  key  返回集合中所有成员

sismembers  key   member  判断member元素是否是集合key的成员

scard key  获取集合里面的元素个数

srem key value  删除集合中指定元素

srandmember key  数值     从set集合里面随机取出指定数值个元素   如果超过最大数量就全部取出，

spop key  随机移出并返回集合中某个元素

smove  key1  key2  value(key1中某个值)   作用是将key1中执定的值移除  加入到key2集合中

sdiff key1 key2  在第一个set里面而不在后面任何一个set里面的项(差集)

sinter key1 key2  在第一个set和第二个set中都有的 （交集）

sunion key1 key2  两个集合所有元素（并集）

#### 4.hash

Redis hash 是一个键值对集合。
Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

kv模式不变，但v是一个键值对

类似Java里面的Map<String,Object>

hset  key  (key value)  向hash表中添加一个元素

hget key  key  向hash表中获取一个元素

hmset  key key1 value1 key2 value2 key3 value3 向集合中添加一个或多个元素

hmget key  key1 key2 key3  向集合中获取一个或多个元素

hgetall  key   获取在hash列表中指定key的所有字段和值

hdel  key  key1 key2 删除一个或多个hash字段

hlen key 获取hash表中字段数量

hexits key key  查看hash表中，指定key（字段）是否存在

hkeys  key 获取指定hash表中所有key（字段）

hvals key 获取指定hash表中所有value(值)

hincrdy key  key1  数量（整数）  执定hash表中某个字段加  数量  ，和incr一个意思

hincrdyfloat key key1  数量（浮点数，小数）  执定hash表中某个字段加  数量  ，和incr一个意思

hsetnx key key1 value1  与hset作用一样，区别是不存在赋值，存在了无效。

#### 5.zset

Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。
不同的是每个元素都会关联一个double类型的分数。
redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的,但分数(score)却可以重复。

zadd  key  score 值   score 值   向集合中添加一个或多个成员

zrange key  0   -1  表示所有   返回指定集合中所有value

zrange key  0   -1  withscores  返回指定集合中所有value和score

zrangebyscore key 开始score 结束score    返回指定score间的值

zrem key score某个对应值（value），可以是多个值   删除元素

zcard key  获取集合中元素个数

zcount key   开始score 结束score       获取分数区间内元素个数

zrank key vlaue   获取value在zset中的下标位置(根据score排序)

zscore key value  按照值获得对应的分数



## redis的持久化机制

在指定的时间间隔内，将内存当中的数据集快照写入磁盘，它恢复时是将快照文件直接读到内存

redis提供两种方式进行持久化
- RDB持久化默认，
- AOF（append only file）持久化。

### 1.RDB

#### 是什么？

原理是redis会单独创建（fork）一个与当前线程一模一样的子进程来进行持久化，这个子线程的所有数据（变量。环境变量，程序程序计数器等）都和原进程一模一样，会先将数据写入到一个临时文件中，待持久化结束了，再用这个临时文件替换上次持久化好的文件，整个过程中，主进程不进行任何的io操作，这就确保了极高的性能

#### 1.这个持久化文件在哪里
配置文件中  dir  指定的位置
#### 2.他什么时候fork子线程，或者什么时候触发rdb持久化机制
- shutdown时，如果没有开启aof，会触发
 
 配置文件中默认的快照配置
 执行命令save或者bgsave  save是只管保存，其他不管，全部阻塞   bgsave： redis会在后台异步进行快照操作，同时可以响应客户端的请求
 执行flushall命令  但是里面是空的，无意义

### 2.aof(--fix)     ls -l --block-size=M

#### 是什么？

原理是将Reids的操作日志以追加的方式写入文件，读操作是不记录的

#### 1.这个持久化文件在哪里

#### 2.触发机制（根据配置文件配置项）

- no：表示等操作系统进行数据缓存同步到磁盘（快，持久化没保证）
- always：同步持久化，每次发生数据变更时，立即记录到磁盘（慢，安全）
- everysec：表示每秒同步一次（默认值,很快，但可能会丢失一秒以内的数据）

#### 3.aof重写机制

当AOF文件增长到一定大小的时候Redis能够调用 bgrewriteaof对日志文件进行重写 。当AOF文件大小大于该配置项时自动开启重写
auto-aof-rewrite-min-size 64mb

当AOF文件增长到一定大小的时候Redis能够调用 bgrewriteaof对日志文件进行重写 。当AOF文件大小的增长率大于该配置项时自动开启重写（这里指超过原大小的100%）。
auto-aof-rewrite-percentage 100


#### 4.redis4.0后混合持久化机制

##### 开启混合持久化

4.0版本的混合持久化默认关闭的，通过aof-use-rdb-preamble配置参数控制，yes则表示开启，no表示禁用，5.0之后默认开启。

混合持久化是通过bgrewriteaof完成的，不同的是当开启混合持久化时，fork出的子进程先将共享的内存副本全量的以RDB方式写入aof文件，然后在将重写缓冲区的增量命令以AOF方式写入到文件，写入完成后通知主进程更新统计信息，并将新的含有RDB格式和AOF格式的AOF文件替换旧的的AOF文件。简单的说：新的AOF文件前半段是RDB格式的全量数据后半段是AOF格式的增量数据，

优点：混合持久化结合了RDB持久化 和 AOF 持久化的优点, 由于绝大部分都是RDB格式，加载速度快，同时结合AOF，增量的数据以AOF方式保存了，数据更少的丢失。

缺点：兼容性差，一旦开启了混合持久化，在4.0之前版本都不识别该aof文件，同时由于前部分是RDB格式，阅读性较差

## 小总结：

### 1.redis提供了rdb持久化方案，为什么还要aof？

优化数据丢失问题，rdb会丢失最后一次快照后的数据，aof丢失不会超过2秒的数据

### 2.如果aof和rdb同时存在，听谁的？

aof

### 3.rdb和aof优势劣势

rdb 适合大规模的数据恢复，对数据完整性和一致性不高 ，  在一定间隔时间做一次备份，如果redis意外down机的话，就会丢失最后一次快照后的所有操作
aof 根据配置项而定

1.官方建议   两种持久化机制同时开启，如果两个同时开启  优先使用aof持久化机制  

### 性能建议（这里只针对单机版redis持久化做性能建议）：

因为RDB文件只用作后备用途，只要15分钟备份一次就够了，只保留save 900 1这条规则。

如果Enalbe AOF，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了。
代价一是带来了持续的IO，二是AOF rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。
只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上。
默认超过原大小100%大小时重写可以改到适当的数值。