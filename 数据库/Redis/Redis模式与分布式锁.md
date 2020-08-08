# Redis模式

## 单点模式

单节点模式是最简单的Redis模式，它采用单个 Redis 节点部署架构，没有备用节点实时同步数据，不提供数据持久化和备份策略，适用于数据可靠性要求不高的纯缓存业务场景。



## 主从复制模式

主从模式就是在N个Redis服务器中，一台Redis服务器作为主节点，其余的作为从节点，主节点的数据复制到其他的Redis服务器，且复制只能由主节点到从节点。Redis主从复制支持主从同步和从从同步

主从模式的一个作用是备份数据，这样当一个节点损坏（指不可恢复的硬件损坏）时，数据因为有备份，可以方便恢复；另一个作用是负载均衡，所有客户端都访问一个节点会影响Redis工作效率，有了主从以后，查询操作就可以通过查询从节点来完成。

默认配置下，master节点可以进行读和写，slave节点只能进行读操作。如果slave节点挂了不会影响其他slave节点的读和master节点的读和写重新启动后会将数据从master节点同步过来；master节点挂了以后，不影响slave节点的读，Redis将不再提供写服务，master节点启动后Redis将重新对外提供写服务。

需要说明的是：master节点挂了以后，slave不会竞选成为master，这是主从模式的一个缺点。

### 演示

**建立复制**

分别在两个cmd下创建两个不同Redis实例

```
redis-server --port 6379
redis-server --port 6380
```

> 此时两个Redis节点默认为主节点。

我们再分别新建两个cmd并连接本地的redis服务，并在6380客户端中执行`slaveof` 命令

```
redis-cli -p 6379
redis-cli -p 6380
# 成为本地 6379 端口实例的从节点
SLAVEOF 127.0.0.1  6379
```

**观察复制效果**

我们在从节点6380中查询一个不存在的key

```
127.0.0.1:6380> GET mkey
(nil)
```

然后在主节点6379中添加这个key

```
127.0.0.1:6379> set mkey mval
OK
```

此时再从 从节点中查询，即可发现该结构

```
127.0.0.1:6380> get mkey
"mval"
```

我们可以在从节点执行slaveof no one来断开复制，此时**从节点不会删除已有数据**，只是不再接受主节点新的数据变化。



### **复制过程**

1. 从节点执行 slaveof 命令，此时从节点只是保存了 slaveof 命令中主节点的信息，并没有立即发起复制；
2. 从节点内部的定时任务发现有主节点的信息，开始使用 socket 连接主节点；
3. 连接建立成功后，发送 ping 命令，希望得到 pong 命令响应，否则会进行重连；
4. 如果主节点设置了权限，那么就需要进行权限验证；如果验证失败，复制终止；
5. 权限验证通过后，进行数据同步，**这是耗时最长的操作**，主节点将把所有的数据全部发送给从节点；
6. 当主节点把当前的数据同步给从节点后，便完成了复制的建立流程。接下来，**主节点就会持续的把写命令发送给从节点，保证主从数据一致性**。



## Sentinel 哨兵模式

该模式分为哨兵和数据两部分，其中哨兵部分由一个或多个哨兵节点组成，哨兵节点是特殊的 Redis 节点，不存储数据；主节点和从节点都是数据节点；

在主从复制的基础上，哨兵实现了 **自动化的故障恢复** 功能，它的功能是：

-  哨兵会不断地检查主节点和从节点是否运作正常。
- 当 **主节点** 不能正常工作时，哨兵会开始 **自动故障转移操作**，它会将失效主节点的其中一个 **从节点升级为新的主节点**，并让其他从节点改为复制新的主节点。
- 客户端在初始化时，通过连接哨兵来获得当前 Redis 服务的主节点地址。
- 哨兵可以将故障转移的结果发送给客户端。



### 演示

**创建主从节点**

在Redis的安装目录下分别新建*`redis-master.conf`/`redis-slave1.conf`/`redis-slave2.conf`*作为1个主节点和2个从节点，并分别写上如下内容：

```
#redis-master.conf 
port 6379 
daemonize yes 
logfile "6379.log" 
dbfilename "dump-6379.rdb" 
#redis-slave1.conf 
port 6380 
daemonize yes 
logfile "6380.log" 
dbfilename "dump-6380.rdb" 
slaveof 127.0.0.1 6379
#redis-slave2.conf 
port 6381 
daemonize yes 
logfile "6381.log" 
dbfilename "dump-6381.rdb" 
slaveof 127.0.0.1 6379

```

然后在3个cmd下分别执行如下语句来启动不同的Redis实例

```
redis-server E:\Redis-x64-4.0.14.2\redis-master.conf
redis-server E:\Redis-x64-4.0.14.2\redis-slave1.conf
redis-server E:\Redis-x64-4.0.14.2\redis-slave2.conf
```

然后在一个新cmd下执行*`redis-cli`*默认连接到端口为 `6379` 的主节点，然后执行 `info Replication` 检查一下主从状态是否正常：

![](https://user-gold-cdn.xitu.io/2020/6/17/172bfd931b43eb05?w=521&h=337&f=png&s=14496)



**创建哨兵节点**

同样的方式我们创建3个哨兵节点，并创建其配置文件：

```
# redis-sentinel-1.conf
port 26379 
daemonize yes 
logfile "26379.log" 
# 该哨兵节点监控主节点 127.0.0.1:6379，主节点名为mymaster
# 数字2表示至少需要 2 个哨兵节点同意，才能判定主节点故障并进行故障转移。
sentinel monitor mymaster 127.0.0.1 6379 2 
# redis-sentinel-2.conf 
port 26380 
daemonize yes 
logfile "26380.log" 
sentinel monitor mymaster 127.0.0.1 6379 2 
# redis-sentinel-3.conf 
port 26381 
daemonize yes 
logfile "26381.log" 
sentinel monitor mymaster 127.0.0.1 6379 2

```

执行如下命令启动哨兵节点

```
redis-server E:\Redis-x64-4.0.14.2\redis-sentinel-1.conf --sentinel
redis-server E:\Redis-x64-4.0.14.2\redis-sentinel-2.conf --sentinel
redis-server E:\Redis-x64-4.0.14.2\redis-sentinel-3.conf --sentinel
```

连接端口为 26379 的 Redis 节点，并执行 `info Sentinel` 命令来查看是否已经在监视主节点：

![](https://user-gold-cdn.xitu.io/2020/6/17/172c003ae351ef06?w=615&h=156&f=png&s=5427)







**演示故障转移**

通过命令*`netstat -aon|findstr 6379`*查询端口对应进程ID（我得到的id为7672），然后杀掉主节点6379

![](https://user-gold-cdn.xitu.io/2020/6/17/172c0069ec1c3827?w=381&h=58&f=png&s=1668)

然后我们再去哨兵节点那里查看信息，刚杀掉主节点时可能主节点尚未切换，发现故障并转移需要一段时间：

![](https://user-gold-cdn.xitu.io/2020/6/17/172c007db758bc88?w=633&h=280&f=png&s=11382)

此时slave仍为2表明哨兵节点认为新的主节点仍然有两个从节点，这时这是因为哨兵在将 `6381` 切换成主节点的同时，将 `6379` 节点置为其从节点。虽然 `6379` 从节点已经挂掉，但是由于 **哨兵并不会对从节点进行客观下线**，因此认为该从节点一直存在。当 `6379` 节点重新启动后，会自动变成 `6381` 节点的从节点。


### 如何挑选新主节点

1. 在失效主节点属下的从节点里，那些被标记为被标记为主观下线、已断线、或最后一次回复 PING 命令的时间大于五秒钟的从节点都会被淘汰。
2. 在失效主节点属下的从节点里，与失效主节点连接断开的时长超过 down-after 选项指定的时长十倍的从节点会被 淘汰。
3. 经过上述两部淘汰剩余的从节点中，选出**复制偏移量最大** 的那个 从节点 作为新的主节点；如果复制偏移量不可用，或者从节点的复制偏移量相同，那么 **带有最小运行 ID** 的那个从节点成为新的主节点。

## 集群模式

Redis 集群是一个提供在**多个Redis间节点间共享数据**的程序集，客户端任意直连到集群中的任意一台，即可对其他Redis节点进行读写操作。

![](https://user-gold-cdn.xitu.io/2020/6/17/172c01c5f1e7b400?w=589&h=392&f=png&s=120153)

### **集群的优点**

- 集群将数据分散到多个节点，**一方面** 突破了 Redis 单机内存大小的限制，**存储容量大大增加**；**另一方面** 每个主节点都可以对外提供读服务和写服务.
- 集群支持主从复制和主节点的 **自动故障转移** ，当任一节点发生故障时，集群仍然可以对外提供服务。

### **基本原理**

Redis 集群中内置了 `16384` 个哈希槽，当客户端连接到 Redis 集群之后，会同时得到一份关于这个 **集群的配置信息**，当客户端具体对某一个 `key` 值进行操作时，会计算出它的一个 Hash 值，然后把结果对 `16384` **求余数**，这样每个 `key` 都会对应一个编号在 `0-16383` 之间的哈希槽，Redis 会根据节点数量 **大致均等** 的将哈希槽映射到不同的节点。如果查询的key不归属于某个Redis节点，那么它会使用特殊的`MOVED` 命令来进行一个跳转，告诉客户端去连接这个节点以获取数据

```
GET x
-MOVED 3999 127.0.0.1:6381
```

如上所示，符号-表示这是一个错误信息，在收到 `MOVED` 指令后，就立即纠正本地的 **槽位映射表**，那么下一次再访问 `key` 时就能够到正确的地方去获取了；3999表示key对应的槽位编号，后面跟着的是目标Redis节点地址。

#### 数据分区的方案

前面我们介绍的是哈希取余的分区思路，但这个方案存在一个问题：**当新增或删减节点时**，节点数量发生变化，系统中所有的数据都需要 **重新计算映射关系**，引发大规模数据迁移。

##### **一致性哈希分区**

该算法将整个空间组成一个虚拟的圆环，根据每一个数据的key值算出其hash值，确定它在圆环上的位置，然后从此位置顺时针行走，找到的第一个节点就是其所属节点。

![](https://user-gold-cdn.xitu.io/2020/6/17/172c03536a59fc09?w=561&h=394&f=png&s=156505)

与哈希取余方案相比，该方案将**增减节点的影响限制在相邻节点**，以上图为例，若在node1和node2之间增加node5，则则只有 `node2` 中的一部分数据会迁移到 `node5`；如果去掉 `node2`，则原 `node2` 中的数据只会迁移到 `node4` 中，只有 `node4` 会受影响。

该方案存在一个问题：当节点数量较少时，增加或删减节点，对单个节点的影响可能很大，造成数据的严重不平衡。比如上图中去掉node2，node4中的数据由总数据的1/4左右变为1/2左右，与其他节点相比负载过高。



##### **带虚拟节点的一致性哈希分区**

这是Redis集群使用的方案。该方案在 一致性哈希分区的基础上，引入了 **虚拟节点（也成为槽） **的概念，每个实际节点包含一定数量的槽，每个槽包含哈希值在一定范围内的数据。引入槽以后，查询数据值的过程变为：

1. 对数据的特征值（一般是key）计算哈希值，使用的算法是CRC16

2. 根据哈希值，计算数据属于哪个槽。

3. 根据槽与节点的映射关系，计算数据属于哪个节点。

   

此时槽是数据管理和迁移的基本单位。槽解耦了数据和实际节点之间的关系，减小了增加或删除节点对系统的影响。

此处我们仍以上图为例，系统中有4个实际节点，假设为其分配16个槽(0-15)；槽0-3位于node1，4-7位于node2，以此类推。如果此时删除node2，只需要将槽4-7重新分配即可，例如槽4-5分配给node1，槽6分配给node3，槽7分配给node4；可以看出删除node2后，数据在其他节点的分布仍然较为均衡。

![](https://user-gold-cdn.xitu.io/2020/6/17/172c03a8d44005f4?w=634&h=357&f=png&s=185008)

> 在Redis集群中，槽的数量为16384。



#### 节点通信机制

##### 两个端口

在 **集群** 中，没有数据节点与非数据节点之分：**所有的节点都存储数据，也都参与集群状态的维护**。为此，集群中的每个节点，都提供了两个 TCP 端口：

- **普通端口**：主要为客户端提供服务；在节点间数据迁移也会使用
- **集群端口**： 端口号是普通端口 + 10000 。它只用于**节点之间的通信**，如搭建集群、增减节点、故障转移等操作时节点间的通信。不要使用客户端连接集群接口

##### Gossip 协议

节点间通信按照通信协议可以分为几种类型：单对单、广播、Gossip 协议等。此处只说明广播和 Gossip 的对比：

- **广播**是指向集群内**所有节点**发送消息。这种做法的优点是集群内所有节点获得的集群信息是一致的，缺点是每条消息都要发送给所有节点，CPU、带宽等消耗较大。
- **Gossip 协议**：在节点数量有限的网络中，每个节点都**根据特定的规则**与**部分节点**通信 。

#### 数据结构

节点需要专门的数据结构来存储集群的状态，此处介绍两个：`clusterNode` 和 `clusterState` 结构：前者记录了一个节点的状态，后者记录了集群作为一个整体的状态。

**clusterNode 结构**

```
typedef struct clusterNode { 
    //节点创建时间
    mstime_t ctime; 
    //节点id 
    char name[REDIS_CLUSTER_NAMELEN]; 
    //节点的ip和端口号 
    char ip[REDIS_IP_STR_LEN]; 
    int port; 
    //节点标识：整型，每个bit都代表了不同状态，如节点的主从状态、是否在线、是否在握手等 
    int flags; 
    //配置纪元：故障转移时起作用，类似于哨兵的配置纪元
    uint64_t configEpoch;
    //槽在该节点中的分布：占用16384/8个字节，16384个比特；每个比特对应一个槽：比特值为1，则该比特对应的槽在节点中；比特值为0，则该比特对应的槽不在节点中 
    unsigned char slots[16384/8]; 
    //节点中槽的数量 
    int numslots;
    ………… 
} clusterNode;

```



**clusterState 结构**

`clusterState` 结构保存了在当前节点视角下，集群所处的状态。主要字段包括：

```
typedef struct clusterState { 
    //自身节点
     clusterNode *myself; 
    //配置纪元 
    uint64_t currentEpoch; 
    //集群状态：在线还是下线
    int state; 
    //集群中至少包含一个槽的节点数量 
    int size; 
    //哈希表，节点名称->clusterNode节点指针 
    dict *nodes; 
    //槽分布信息：数组的每个元素都是一个指向clusterNode结构的指针；如果槽还没有分配给任何节点，则为NULL 
    clusterNode *slots[16384]; 
    ………… 
}clusterState;
```





# 分布式锁

## 什么是分布式系统

**什么是集中式系统**

在学习分布式之前，先了解一下与之相对应的集中式系统是什么：即一个主机带多个终端。终端没有数据处理能力，仅负责数据的录入和输出。而运算、存储等全部在主机上进行。集中式系统主要流行于上个世纪。

**（1）集中式系统优点**

集中式系统的最大的特点就是部署结构非常简单，底层一般采用从IBM、HP等厂商购买到的昂贵的大型主机。无需考虑如何对服务进行多节点的部署，无需考虑各节点之间的分布式协作问题。

**（2）集中式系统缺点**

由于采用单机部署。很可能带来系统大而复杂、难于维护；发生单点故障而导致整个系统或者网络的瘫痪。

**什么是分布式系统**

简单来说就是一群独立计算机集合共同对外提供服务，但是对于系统的用户来说，就像是一台计算机在提供服务一样。分布式意味着可以采用更多的普通计算机（相对于昂贵的大型机）组成分布式集群对外提供服务。计算机越多，CPU、内存、存储资源等也就越多，能够处理的并发访问量也就越大。

各个主机之间通信和协调主要通过网络进行，所以，分布式系统中的计算机在空间上几乎没有任何限制，这些计算机可能被放在不同的机柜上，也可能被部署在不同的机房中，还可能在不同的城市中，对于大型的网站甚至可能分布在不同的国家和地区



## 为什么需要分布式锁

我们在系统中修改已有数据时，需要先读取再修改保存，由于修改和保存不是原子操作，在并发场景下，部分对数据的操作可能会丢失。在单服务器系统我们常用本地锁（synchronized或ReetrantLock）来避免并发带来的问题，然而，当服务采用集群方式部署时，本地锁无法在多个服务器之间生效，这时候保证数据的一致性就需要分布式锁来实现。



## 分布式锁的特点

（1）分布式锁需要保证在不同节点的不同线程的互斥。

（2）同一个节点上的同一个线程如果获取了锁之后那么也可以再次获取这个锁。

（3）锁超时:和本地锁一样支持锁超时，防止死锁。

（4）支持阻塞和非阻塞:和ReentrantLock一样支持lock和trylock以及tryLock(long timeOut)。



## 常见的分布式锁

一般实现分布式锁有以下几个方式:

- MySql
- Redis
- ZooKeeper



# Redis的分布式锁

Redis锁主要利用了setnx命令：

```
> exists job				# job 不存在
(integer) 0
> setnx job "learn"			#如果键不存在，对键进行设置操作并返回1，否则返回0。
(integer) 1
> setnx job "test"			# 尝试覆盖 job ，失败
(integer) 0
> get job
"learn"
```

解锁命令：DEL key，通过删除键值对释放锁，以便其他线程可以通过 SETNX 命令来获取锁。

锁超时：EXPIRE key timeout, 设置 key 的超时时间，以防止锁没有被显示的释放。



## 分布式锁的问题

### **死锁问题**

如果 SETNX 成功，在设置锁超时时间后，服务器挂掉、重启或网络问题等，导致 EXPIRE 命令没有执行，锁没有设置超时时间变成死锁。

针对此场景，Redis提供了lua脚本支持。用户可以向服务器发送 lua 脚本来执行自定义动作，获取脚本的响应数据。Redis 服务器会单线程原子性执行 lua 脚本，其他机器的所有命令都必须等到Lua脚本执行结束才能执行。

**EVAL命令**

命令格式为：

```lua
EVAL script numkeys key [key …] arg [arg …]
```

- `script`参数是一段 Lua5.1 脚本程序
- `numkeys`指定后续参数有几个key
- `key [key …]`表示在脚本中所用到的Redis 键，可以在Lua脚本中通过KEYS[1], KEYS[2]获取。
- `arg [arg …]` 附加参数。在Lua脚本中通过ARGV[1],ARGV[2]获取

**示例**

```
# 示例1:只有1个key，args数组无元素
127.0.0.1:6379> EVAL "return KEYS[1]" 1 key1
"key1"

# 示例2：keys数组无元素，arg数组元素中有1个元素value1
127.0.0.1:6379> EVAL "return ARGV[1]" 0 value1
"value1"

# 使用redis为lua内置的redis.call函数
# 脚本程序先执行setnx命令，再执行expire命令
# key数组里的key表示redis的key；
# args数组里的value和100分别对应redis的value和100。此处redis使用lua的tonumber将字符串转换为数字
EVAL "if (redis.call('setnx',KEYS[1],ARGV[1]) < 1) then return 0; end; redis.call('expire',KEYS[1],tonumber(ARGV[2])); return 1;" 1 key value 100
```



### 锁误解除

如果线程A将锁设置30秒过期，那么当过了30秒后，锁自动释放，被另一线程B获取。此时A执行完成后，使用DEL释放锁，那么此时线程A释放的是线程B的锁。

解决方法是生成一个 UUID 标识当前线程，在删除之前验证 key 对应的 value 判断锁是否是当前线程持有。

```lua
String uuid = UUID.randomUUID().toString().replaceAll("-","");

# Redis在 2.6.12 版本开始，为 SET 命令增加一系列选项：
#      SET key value[EX seconds][PX milliseconds][NX|XX]
# 其中EX seconds，PX milliseconds都表示过期时间，前者单位是秒，后者为毫秒。XX表示仅当key存在时设置值
# 设置 key-uuid，NX表示仅当key不存在时设置值，EX 30表示在30秒后删除

SET key uuid NX EX 30
// 解锁
if (redis.call('get', KEYS[1]) == ARGV[1])
    then return redis.call('del', KEYS[1])
else return 0
end
```

这种方式在Redis集群模式下可能会出现问题：客户端A在Redis的master节点上拿到了锁，但是这个加锁的key还没有同步到slave节点。此时如果master故障，发生故障转移，一个slave节点升级为master节点，B客户端也可以获取同个key的锁，但客户端A已经是拿到锁了，这就导致多个客户端都拿到锁。

## Redlock

针对分布式环境，Redis作者提出了一种更高级的分布式锁Redlock。

之前我们介绍过Redis在java的应用：jedis，此处再介绍另一种：Redisson。Jedis是阻塞式I/O，而Redisson底层使用Netty可以实现非阻塞I/O，并且它封装了锁，因此我们可以像ReentrantLock一样使用Redisson。

**示例**

使用前需导入依赖：

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.10.6</version>
</dependency>
```

使用Redisson：

```java
// 配置文件
Config config = new Config();
config.useSingleServer()
        .setAddress("redis://127.0.0.1:6379")
        .setPassword(RedisConfig.PASSWORD)
        .setDatabase(0);
//构造RedissonClient
RedissonClient redissonClient = Redisson.create(config);

// 设置锁定资源名称
RLock lock = redissonClient.getLock("redlock");
lock.lock();
try {
    System.out.println("获取锁成功，实现业务逻辑");
    Thread.sleep(10000);
} catch (InterruptedException e) {
    e.printStackTrace();
} finally {
    lock.unlock();
}
```



# 参考资料

[Redis(9)——史上最强【集群】入门实践教程](https://www.wmyskxz.com/2020/03/17/redis-9-shi-shang-zui-qiang-ji-qun-ru-men-shi-jian-jiao-cheng/#toc-heading-25)

[大家都在说的分布式系统到底是什么？](https://juejin.im/post/5af8ea34f265da0b9f40622a)

[Redis分布式锁]([https://pjmike.github.io/2019/04/25/%E5%9F%BA%E4%BA%8ERedis%E7%9A%84%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81%E5%AE%9E%E7%8E%B0/](https://pjmike.github.io/2019/04/25/基于Redis的分布式锁实现/))