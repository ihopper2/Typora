## 1.概述

### 1.1 什么是Redis

Redis是一个使用C语言编写的开源的，高性能的非关系型数据库。

Reids可以存储键和五种不同类型值之间的映射，键只可以是字符串，值支持五种类型：字符串、列表、集合、散列表和有序集合。

与传统的数据库不同的是Redis的数据是存储在内存中，所以是读写速度很快，因此Reids可以被广泛应用在缓存方向。每秒可以处理超过10万次读写操作。

### 1.2 Redis有哪些优缺点

优点：

- 读写性能优异，读的速度11万次/s，写的速度8万次/s。
- 支持数据持久化，支持AOF和RDB两种持久化方式。
- 支持事务，Redis所有操作都是原子操作，同时Redis还支持几个操作之后原子性执行。
- 数据结构丰富，除了支持String类型的value，还有list，set，hash，zset。
- 支持主从复制，读写分离。

缺点：

- 由于是内存数据库，所以，单台机器，存储的数据量，跟机器本身的内存大小。虽然redis本身有key过期策略，但是还是需要提前预估和节约内存。如果内存增长过快，需要定期删除数据。
- 如果进行完整重同步，由于需要生成rdb文件，并进行传输，会占用主机的CPU，并会消耗现网的带宽。不过redis2.8版本，已经有部分重同步的功能，但是还是有可能有完整重同步的。比如，新上线的备机。
- 修改配置文件，进行重启，将硬盘中的数据加载进内存，时间比较久。在这个过程中，redis不能提供服务

### 1.3 为什么要使用Redis

主要是高性能和高并发

高性能：从内存中读取数据比从缓存中读取数据要快。

高并发：直接操作缓存能够承受的请求是远远大于直接访问数据库的。

### 1.4 Redis为什么这么块

- 完全基于内存，绝大部分请求是纯粹的内存操作，非常快速。
- 数据结构简单，对数据操作也简单，Redis中的数据结构是专门设计的。
- 采用单线程，避免了不必要的上下文切换和竞争。
- 使用多路复用IO模型。
  - 多路复用是指使用一个线程来检查多个文件描述符（Socket）的就绪状态，比如调用select和poll函数，传入多个文件描述符，如果有一个文件描述符就绪，则返回，否则阻塞直到超时。得到就绪状态后进行真正的操作可以在同一个线程里执行，也可以启动线程执行（比如使用线程池）

## 2.数据类型

| 数据类型 | 可以存储的值           | 操作                                                         | 数据结构                                                     | 应用场景                                                     |
| -------- | ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| String   | 字符串、整数、浮点数   | 对整个字符串或者字符串一部分执行操作<br/> 对整数和浮点数执行自增和自减操作 | 采用**简单动态字符串**，是可以修改字符串的内瓤，内部类似于ArrayLIst，采用预分配冗余空间的方式减少内存的频繁分配 | 做简单的键值对缓存                                           |
| List     | 列表                   | 从两端压入压入或者弹出元素，对单个或者多个元素进行修建，只保留一个范围内元素 | 使用的quickList，首先数据较少的时候，使用一块连续的内存存储，结构为ziplist，但数据较多的时候才会改成quickList，Redis将链表和zipList结合起来成为quickList，也就是使用双向指针将其串联起来。 | 存储一些列表结构，类似粉丝列表、文章评论列表的数据           |
| Set      | 无序集合               | 添加、获取、移除单个元素，检查一个元素是否存在于集合中。计算交集并集差集 | dict字典，也就是map                                          | 交集、并集、差集的操作，比如交集，可以把两个人的粉丝列表整一个交集。 |
| hash     | 包含键值对的无序散列表 | 添加、获取、移除单个键值对<br/>获取所有键值对<br/>检查某个键是否存在 | 少量ziplist，多量使用hashtable                               | 结构化数据，一个对象                                         |
| zset     | 有序集合               | 添加、获取、删除元素<br/>根据分值范围或者成员来获取元素<br/>计算一个键的排名 | 使用了两个数据结构，hash是为了关联value-socre；跳跃表是为了给元素排序 | 去重但是排序，如获得排名前几的用户                           |

### 2.1 String

set<key> <value>

get<key>

mset<key1><value1><key2><value2><key3><value3>

mget<key1><key2>

setnx<key1><value1>(key1不存在的时候添加)

msetnx<key1><value1><key2><value2><key3><value3>(key不存在的时候添加)

append<key> <value>(给存在的key添加字符串)

strlen<key>求字符串长度

getRange<key>

decr decrby incr incrby

### 2.2 List

lpush/rpush<key><v1><v2><v3><v4>

lpop/rpop<key>

lrange<key><start><end>

llen<key>

linsert<key>before/after<value><v1><v2><v3><v4>

### 2.3 Set

sadd<key><v1><v2><v3><v4>

semembers<key>取出所有元素

sisember<key><value>判断元素是否存在

scard<key> 集合中元素个数

srem<key><v1><v2><v3><v4> 删除集合中元素

spop<key> n 随机吐出几个元素

sinter<key1><key2> 取交集

sunion<key1><key2> 取并集

sdiff<key1><key2> 取key1中有，key2中没有的

### 2.4 hash

hset<key><field1><v1><field2><v2><field3><v3>

hget<key><field>

hexist<key><field>

hkeys<key>列出所有的field

hval<key>列出所有的value

### 2.5 zset

将set中的s换成z

### 2.6 Bitmaps

这个数据类型可以实现对位操作，实际是一个字符串，默认每一位都是0。我们提供了单独的一套命令。以“位”作为单位。每个数组的每个单元只能存储0或者1，数组的下表称为偏移量。

setbit<keys><offset><value>(offset的值要小于2的32次方)

getbit<keys><offset> 获取某一个偏移量的值

bitcount<keys> 统计字符串中为1的个数

## 3.持久化

### 3.1 什么是持久化

就是把内存中的数据读取到磁盘中，防止服务器宕机，内存中失去数据。

### 3.2 RDB持久化

在指定的时间间隔内将内存中的数据快照如磁盘中，对应产生的文件dump.rdb。通过配置文件中的save参数来定义快照周期。

![image-20210815173316662](https://kangrui-pictures.oss-cn-beijing.aliyuncs.com/img/image-20210815173316662.png)

**优势**

- 只有一个dump.rdb文件，方便持久化
- 容灾型好，一个文件就可以保存到安全磁盘
- 性能最大化，fork出子进程完成写操作，让主进程继续处理命令。
- 适合大规模数据集，比AOF的启动效率高。

**劣势**

- 数据安全性低，RDB是间隔一段时间进行持久化，如果持久化期间发生故障，就会造成数据丢失
- Fork时，内存中的数据被克隆了一份，导致大约2倍的碰撞性。

**备份如何进行的**

redis会单独创建出一个子进程进行持久化，会先将数据写入一个临时文件，等到持久化结束了，再用这个临时文件代替上次持久化好的文件，真个过程中，主线程不进行任何IO操作，确保了极高的性能。如果对于数据不是很敏感，进行大规模数据恢复，RDB比AOF效率高。

**FORK**

Fork的作用是复制出了一个和主线程相同的线程（java栈，程序计数器，变量），后来为处于效率的考量，引入了写时复制技术。

### 3.3 AOF持久化

以日志的形式记录每一个写操作，将redis执行过的命令都保存下来，只许追加文件，不能改写文件。

redis启动之初会读取该文件重构数据库。

![image-20210815174552122](https://kangrui-pictures.oss-cn-beijing.aliyuncs.com/img/image-20210815174552122.png)

**优点**

- 数据安全，每进行一次写操作就会记录到文件中一次。
- 即使中途宕机，也能解决数据一致性的问题
- 拥有rewrite模式，当数据过多的时候，会重写，删除某些命令，保存能够重建数据库的命令

**缺点**

- AOF文件比RDB文件大，且恢复效率慢
- 数据集大的时候，启动慢

appendfsync always 每一次     everysec 每一秒

**优缺点是什么**？

- AOF文件比RDB更新频率高，优先使用AOF还原数据。
- AOF比RDB更安全也更大
- RDB性能比AOF好
- 如果两个都配了优先加载AOF

### 3.4 如何选择合适的持久化方式

都开启

## 4.过期键的删除策略

我们可以在Redis中缓存key的过期时间

### 4.1 Redis的过期键的删除策略

- 定时过期，到了时间，自动清除，对内存很友好，但是会占用大量的cpu资源去处理过期数据，从而影响缓存的响应时间和吞吐量。
- 惰性过期，只有访问一个key的时候，才会判断它是否过期，过期则清除。该策略对cpu很友好，但是对于内存很不友好，极有可能出大量过期key占用内存。
- 定期过期，每个一段时间扫描一次数据库的key，过期的清除。折中方案，可以达到cpu和内存的平衡。

Redis中同时使用惰性过期和定期过期。

### 4.2 Redis中过期时间和永久有效怎么设置

expire persist

## 5.内存相关

### 5.1 Mysql中有2000万数据，redis中只有20万数据，如何保证redis中的数据都是热点数据

redis中的数据达到一定量，就会实施淘汰策略。

### 5.2 Redis的内存淘汰策略有哪些

淘汰策略指的是当缓存内存不足时，怎么处理新的写入申请且需要额外空间

**全局的键空间选择移除**

- noeviction：当内存不足时，写入会报错
- allkeys-lru：当内存不足时，移除用的最少的键
- allkeys-random：当内存不足时，随机移除

**设置过期时间的键空间选择移除**

- volatile-lru：当内存不足时，在设置了过期时间的键中，选择移除使用最少的key
- volatile-random：当内存不足时，在设置了过期时间的键中，随机移除key
- volatile-ttl：当内存不足时，在设置了过期时间的键中，移除最早过期时间的key

总结：淘汰策略不会影响到过期key的处理，内存淘汰策略处理的是内存不足时需要额外的新空间；过期策略时处理过期的key。

### 5.3 Redis主要消耗什么物理资源

内存

### 5.4 Redis的内存用完了会怎么样

写入会报错

配合内存淘汰机制，冲刷旧内容

### 5.5 Redis如何做优化

可以好好利用五种基本数据，通常情况下我们可以让key-value使用更加紧凑的方式，尽可能地使用散列表。散列表使用地内存很小。比如我们可以在web中将一个对象地信息的放到hash中，而不是每一个都设置一个key

## 6.事务

### 6.1 什么是事务

事务是一个单独地隔离操作，事务中所用命令都序列化，顺序地执行。事务在执行地时候，不会被其他地任务所打断。

事务是一个原子操作，事务地命令要么都执行，要么都不执行

### 6.2 Redis事务的概念

Redis事务的本质是通过MULTI、EXEC、WATCH等一组命令的集合。

事务支持一次执行多个命令，一个事务中所有命令都会被序列化。在事务执行过程，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中。

总结说：redis事务就是一次性、顺序性、排他性的执行一个队列中的一系列命令。

### 6.3 Redis事务的三个阶段

事务开始 MULTI
命令入队
事务执行 EXEC
事务执行过程中，如果服务端收到有EXEC、DISCARD、WATCH、MULTI之外的请求，将会把请求放入队列中排队

### 6.4 Redis事务相关命令

Redis事务功能是通过MULTI、EXEC、DISCARD和WATCH 四个原语实现的

Redis会将一个事务中的所有命令序列化，然后按顺序执行。

redis 不支持回滚，“Redis 在事务失败时不进行回滚，而是继续执行余下的命令”， 所以 Redis 的内部可以保持简单且快速。
如果在一个事务中的命令MULTI出现错误，那么所有的命令都不会执行；
如果在一个事务中出现运行错误，那么正确的命令会被执行。

- WATCH 命令是一个乐观锁，可以为 Redis 事务提供 check-and-set （CAS）行为。 可以监控一个或多个键，一旦其中有一个键被修改（或删除），之后的事务就不会执行，监控一直持续到EXEC命令。
- MULTI命令用于开启一个事务，它总是返回OK。 MULTI执行之后，客户端可以继续向服务器发送任意多条命令，这些命令不会立即被执行，而是被放到一个队列中，当EXEC命令被调用时，所有队列中的命令才会被执行。
- EXEC：执行所有事务块内的命令。返回事务块内所有命令的返回值，按命令执行的先后顺序排列。 当操作被打断时，返回空值 null。
- 通过调用DISCARD，客户端可以清空事务队列，并放弃执行事务， 并且客户端会从事务状态中退出。
- UNWATCH命令可以取消watch对所有key的监控。

### 6.5 Redis事务支持隔离性吗

Redis 是单进程程序，并且它保证在执行事务时，不会对事务进行中断，事务可以运行直到执行完所有事务队列中的命令为止。因此，Redis 的事务是总是带有隔离性的。

### 6.6 Redis事务保证原子性吗，支持回滚吗

Redis中，单条命令是原子性执行的，但事务不保证原子性，且没有回滚。事务中任意命令执行失败，其余的命令仍会被执行。

## 7.缓存异常

### 7.1 缓存穿透

缓存穿透是指**缓存和数据库中都没有的数据**，导致所有的请求都落到数据库上，造成数据库短时间内承受大量请求而崩掉。

解决方案

- 接口层增加校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截；
- 从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个id暴力攻击
- 采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的 bitmap 中，一个一定不存在的数据会被这个 bitmap 拦截掉，从而避免了对底层存储系统的查询压力

### 7.2 缓存击穿

缓存中没有，数据库中有，导致大量的请求落到了数据库上

解决方案

- 预选设置热门数据
- 设置热点数据永远不过期。
- 加互斥锁

### 7.3 缓存雪崩

指缓存同一时间大面积的失效，所以，后面的请求都会落到数据库上，造成数据库短时间内承受大量请求而崩掉。

**解决方案**

- 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。

- 一般并发量不是特别多的时候，使用最多的解决方案是加锁排队。
- 给每一个缓存数据增加相应的缓存标记，记录缓存的是否失效，如果缓存标记失效，则更新数据缓存。

## 8.集群方案

### 8.1 哨兵模式

![image-20210816074948513](https://kangrui-pictures.oss-cn-beijing.aliyuncs.com/img/image-20210816074948513.png)

**哨兵介绍**

哨兵是集群中非常重要的一个组件，主要有以下功能：

- 集群监控：负责监控master和slave进程是否正常
- 消息通知：某个redis若是有故障，那么哨兵负责发送消息给管理人员
- 故障转移：如果master node挂了，自动转移到slaver node上
- 配置中心：如果故障发生了转移，通知client客户端新的master地址

哨兵用于实现redis集群的高可用，本身也是分布式的，作为一个哨兵集群去运行，互相协同工作。

- 故障转移时，判断一个master node是否宕机，需要大部分哨兵同意才可以，涉及到了分布式选举的问题。
- 即使部分哨兵挂了，集群还是可以正常工作的。

**哨兵核心知识**

- 哨兵至少需要三个实例
- 哨兵+主从架构模型，不能保证数据的不丢失

### 8.2 官方Redis Cluster 方案(服务端路由查询)

Redis没有使用一致性hash，而是使用槽的概念，一共分成16384个槽，将请求发送到任意节点，接收到请求的节点会将查询请求发送到任意节点。

## 9.主从复制

主机将数据更新后根据配置和策略自动同步到备用的master/slave机制。master负责写，slave负责读。

读写分离，性能扩展。

容灾快速恢复

### 9.1 一主二仆

**切入点问题？slave1、slave2挂了，是从头开始复制还是从切入点开始复制?比如从k4进来，那之前的k1,k2,k3是否也可以复制？**

可以复制

**从机是否可以写？set可否？**

不可以

**主机shutdown后情况如何？从机是上位还是原地待命？**

原地待命

**主机又回来了后，主机新增记录，从机还能否顺利复制？**



可以复制

**其中一台从机down后情况如何？依照原有它能跟上大部队吗？**

可以

### 9.2 薪火相传

上一个Slave可以是下一个slave的Master，Slave同样可以接收其他 slaves的连接和同步请求，那么该slave作为了链条中下一个的master, 可以有效减轻master的写压力,去中心化降低风险。

用 slaveof  <ip><port>

中途变更转向:会清除之前的数据，重新建立拷贝最新的

风险是一旦某个slave宕机，后面的slave都没法备份

主机挂了，从机还是从机，无法写数据了


### 9.3 反客为主

当一个master宕机后，后面的slave可以立刻升为master，其后面的slave不用做任何修改。

用 slaveof  no one  将从机变为主机。

### 9.4 复制原理

- slave启动之后会给master发送一个sync（数据同步）消息。
- master接收消息之后，把master进行持久化，生成rdb文件，把rdb文件发送给slave，slave拿到rdb文件后进行写入操作。
- 之后当master跟新之后。slave进行增量复制。

## 10.其他问题

### 10.1 Redis相比Memached的区别

- 前者支持的数据类型更加丰富，后者只支持key-value
- redis的速度比后者快
- redis可以持久化数据
- redis使用单线程+io多路复用技术，后者采用多线程+非堵塞io

### 10.2 如何保证缓存数据和数据库数据双写时的数据一致性

- 先删除缓存，在更新数据库，同步到缓存
- 读请求和写请求串行化，但是会导致吞吐量下降

### 10.3 一个字符串类型的值能存储的最大容量为

512M

### 10.4 Redis如何做到大量数据的插入

使用pipe mode的新模式用于执行大量数据的插入

### 10.5 假设Redis中有1亿个key，其中10万个key是以固定的已知前缀开头的，如何找出来

使用命令keys可以扫描出指定模式的key列表

如果redis正在给线上业务提供服务，使用keys指令有什么问题？

redis是单线程的，会导致线程堵塞一段时间，线上服务会停滞一段时间，直到指令执行结束，服务才能恢复。

使用scan可以无堵塞的提取出指定模式key列表，但是会有一定的重复概率，我们可以去重就好了。

### 10.6 使用Redis做过异步队列吗，如何实现的

使用list保存数据，rpush生产消息，lpop消费消息。当lpop没有消息的时候，可以sleep一段时间，然后在检查有没有消息。

### 10.7 Redis如何实现延时队列

使用sortedset，使用时间戳做score，消息内容作为key，调用zadd用来生产消息，消费者使用zrangebyscore获取n秒之前做过轮询的消息。

### 10.8 Redis回收进程如何工作

- 客户执行新的命令，添加了新的数据
- Redis检查内存使用情况，超过设定的内存大小，根据设定好的策略进行回收。
- 一个新的命令被执行
- 所以我们不断地穿越内存限制的边界，通过不断达到边界然后不断地回收回到边界以下。

### 10.10 Redis回收使用什么算法

LRU算法（least recently used 最近最少使用）

根据历史访问记录进行数据淘汰，核心思想，如果近期被使用过，则将来访问的频率也会变高

![image-20210816094027082](https://kangrui-pictures.oss-cn-beijing.aliyuncs.com/img/image-20210816094027082.png)

1. 最开始时，内存空间是空的，因此依次进入A、B、C是没有问题的
2. 当加入D时，就出现了问题，内存空间不够了，因此根据LRU算法，内存空间中A待的时间最为久远，选择A,将其淘汰
3. 当再次引用B时，内存空间中的B又处于活跃状态，而C则变成了内存空间中，近段时间最久未使用的
4. 当再次向内存空间加入E时，这时内存空间又不足了，选择在内存空间中待的最久的C将其淘汰出内存，这时的内存空间存放的对象就是E->B->D

