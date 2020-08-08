## 前提
### 关系型数据库
关系数据库是指采用了关系模型来组织数据的数据库，它通过行（记录）和列（字段）的形式存储数据，这些一系列的行和列被称为表，一个表就是一个关系，而一组表组成了数据库。

**优点**

* 通过事务处理保持数据的一致性
* 支持SQL语言，可以进行 JOIN 等复杂查询
* 相比于其他模型，它使用的二维表结构贴近于现实世界，易于理解

**缺点**

* 当数据量达到一定规模时，由于关系型数据库的系统逻辑非常复杂，为了维护一致性，使得其非常容易发生死锁等的并发问题，导致其读写速度下滑严重。
* 当有较大的结构变更时 表结构更新就会变得比较复杂
* 当访问网站的用户并发性非常高，此时当读写请求数量非常高，对于传统关系型数据库来说，硬盘I/O是一个很大的瓶颈。

### 非关系型数据库

关系型数据在一些数据敏感的应用中表现了糟糕的性能，例如为巨量文档创建索引、高流量网站的网页服务等。因此关系型数据库主要用于执行规模小而读写频繁，或者大批量极少写访问的事务。

非关系型数据库（NoSQL）是对不同于传统的关系数据库的数据库管理系统的统称。相比起前者，它提出了很多不同于前者的理念：

（1）以键值对存储，结构不固定；

（2）每一个元组（记录）可以有不一样的字段，且可以根据需要增加键值对，因此其数据结构不局限于固定的表格模式，并且不使用SQL作为查询语言。


**非关系型数据库分类**

非关系型数据库可以分为如下几类：

（1）面向高性能并发读写的 key-value 数据库：Redis，Tokyo Cabinet属于这一类型

（2）面向海量数据访问的面向文档数据库：它可以在海量的数据中快速的查询数据，典型代表为 MongoDB 以及 CouchDB

（3）面向可扩展性的分布式数据库：这种数据库希望解决传统数据库中可扩展性上的缺陷，它可以适应数据量的增加以及数据结构的变化。

**非关系型数据库优点**

（1）读写性能：比如redis，它无需经过 SQL 层的解析，且是纯内存操作，因此每秒可以处理超过10万次读写操作；

（2）存储格式多：支持key-value形式、文档形式、图片形式，而关系型数据库则只支持基础类型；

（3）简单的扩展：基于键值对，数据没有耦合性（即关联性），容易扩展。

**非关系型数据库缺点**

（1）支持的特性不够丰富：现有产品所提供的功能都比较有限，大多数 NoSQL 数据库都不支持事务

（2）现有产品的不够成熟：大多数产品都还处于初创期，而关系型数据库已经完善了几十年。

## 什么是redis

Redis是一个使用c语言编写，开源的高性能非关系型的键值对数据库。传统的数据库的读写都是从磁盘里取出数据，而Redis的数据都是存在内存中，读写速度非常快，因此Redis被广泛应用于 缓存 方向，此外还经常作为分布式锁。

>不用安装redis即可尝试redis命令：http://try.redis.io/

**优点**

- 读写性能优异
- 支持数据持久化
- 支持事务
- 数据结构丰富
- 支持主从复制，主机会自动将数据同步到从机，可以进行读写分离。

**缺点：**

- 容量受物理内存的限制，因而不能用作海量数据的高性能读写

- 不具备自动容错和恢复功能：主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的 IP 才能恢复。而切换IP后可能出现数据不一致的问题。

-  较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂

**为什么使用redis作为缓存：**

相比于传统数据库，Redis的读写速度非常快，并且在高并发的情况下可以分担数据库的压力。

> 但是使用内存进行数据存储的开销也是较大的，因此我们只会使用Redis存储一些常用和主要的数据，比如用户登录的信息等。

### 缓存问题

**缓存穿透**

查询一个不存在的数据，这个不存在的数据每次请求都要到数据库查询，进而造成缓存穿透。
解决办法是（1）把不存在的空对象也放到缓存里，并设置一个较短过期时间；（2）使用布隆过滤器。

**缓存击穿**

在平常高并发的系统中，大量的请求同时查询一个 key 时，此时这个key正好失效了，就会导致大量的请求都打到数据库上面去。

解决办法是在请求中使用互斥锁。

**缓存雪崩**

当某一时刻发生发生大规模的缓存失效的情况（比如缓存服务宕机），就会有大量的请求进来直接打到数据库上，导致其挂掉。

解决办法是（1）使用集群缓存，比如redis的集群模式。（2）在雪崩发生后开启redis持久化机制，尽快恢复缓存集群

**双写一致性问题**

由于更新Redis和更新数据库的操作不是原子性的，并且在高并发下还会有顺序未知的读取操作。这些问题导致了在更新操作下，可能会出现数据不一致的问题。

解决方法是利用消息队列来实现消息最终一致性的保证，并且还需要消息队列的重试机制保证能更新成功。

### **Redis为什么是单线程**

（1）Redis的数据结构并不全是简单的Key-Value，还有list，hash等复杂的结构，这些结构有可能会进行很细粒度的操作，比如在很长的列表后面添加一个元素，在hash当中添加或者删除一个对象。在单线程的情况下，无需考虑各种锁的问题。

（2）在采用单线程的情况下，避免了不必要的上下文切换

（3）Redis 采用网络IO多路复用技术来保证在多连接的时 系统的高吞吐量。

> 多路复用主要有三种技术：select，poll，epoll。所谓的多路是指多个网络连接，复用则是复用同一个线程。复用技术可以让单个线程高效的处理多个连接请求（尽量减少网络IO的时间消耗），且Redis在内存中操作数据的速度非常快。因此Redis具有很高的吞吐量。



### **Redis的bigkey**

在Redis中，一个字符串最大512MB，一个二级数据结构(例如hash、list、set、zset)可以存储大约40亿个(2^32-1)个元素，但是实际使用一定要防止出现bigkey：

- 对于字符串类型，它的big体现在单个value值很大
- 对于哈希、列表、集合、有序集合：它们的big体现在元素个数太多。

bigkey是怎么产生的：

- 例如按天数存储网站的用户集合
- 将数据从数据库load出来序列化放到Redis里。这种方式虽然常用，但需要注意的是：是否有必要把所有字段都缓存；有没有像管理的数据

bigkey问题会造成一些危害：

- 内存空间不均匀：这样会不利于集群对内存的统一管理，存在丢失数据的隐患。
- 超时阻塞：由于Redis单线程的特性，操作bigkey的通常比较耗时，也就意味着阻塞Redis可能性越大，这样会造成客户端阻塞或者引起故障切换
- 网络拥塞：bigkey也就意味着每次获取要产生的网络流量较大



## Redis的数据结构

Redis的存储是key-value的形式的，其中key是字符串，而value可以是string、list、hash、set、sortset类型。

Redis内部使用ReadisObject对象来表示所有的key和value。

```c
typedef struct redisObject {
    unsigned type:4;        //对象类型  
    unsigned encoding:4;    //对象编码格式
    unsigned lru:REDIS_LRU_BITS; 
    int refcount;
    void *ptr;              // 指向底层实现数据结构的指针
} robj;
```


### 字符串
Redis使用sdshdr结构来表示字符串对象，并没有直接使用C语言的字符串。

```
struct sdshdr {
    int len;
    int free;   //存储字节数组中未使用的字节数量
    char buf[]; //字节数组中会有\0结束符，该结束符不会记录在len
};
```

**SDS与C的字符串的比较**

（1）sdshdr数据结构中用len属性记录了字符串的长度，获取字符串长度只需O(1)

（2）sds内部实现了动态扩展机制：若修改SDS时，空间不足。先会扩展空间，再进行修改

（3）sds内部有空间预分配机制：在扩展空间时，除了分配修改时所必要的空间，还会分配额外的空闲空间

（4）C 语言字符串只能保存 `ascii` 码，对于图片、音频等信息无法保存，SDS 是二进制安全的，写入什么读取就是什么，不做任何过滤和限制；

**常用命令**

```
SET key value       		设置指定key值，它会覆盖已存在key对应的的value
GET key             		获取指定key值
EXISTS key          		是否存在键值对
DEL key
MGET key1 key2 key3  		批量返回三个key对应的value
MGET key1 val1 key2 val2  	设置两个键值对
EXPIRE keyname 3    		设置keyname键值对 3s后过期
```



### 链表 

Redis的链表是无环双向链表，通过void *指针保存不同类型的节点值


redis使用listNode表示节点
``` 
typedef strcut listNode{

    //前置节点
    strcut listNode  *pre;

    //后置节点
    strcut listNode  *next;

    //节点的值
    void  *value;

}listNode
```
list表示链表
```
typedef struct list{

    //表头结点
    listNode  *head;

    //表尾节点
    listNode  *tail;

    //链表长度
    unsigned long len;

    //节点值复制函数
    void *(*dup) (viod *ptr);

    //节点值释放函数
    void  (*free) (viod *ptr);

    //节点值对比函数
    int (*match) (void *ptr,void *key);

}list
```

**使用示例**

```
LPUSH list A  		#向list头部添加一个新元素，尾部则是RPUSH
LRANGE list 0 3  	#从list取出[0,3]范围的value值。其中0 -1表示取出全部元素
LINDEX list 2 		#从list取出下标2的value值
```



### 集合set

Redis的集合是无序，唯一的，相当于Java的HashSet。

整数集合(intset)是set集合的底层数据结构之一，当一个Set集合只包含整数元素且数量不多时，redis就会采用整数集合(intset)作为set(集合)的底层实现。

整数集合保证了元素是不会出现重复的，并且是从小到大的序列，其结构如下：

```
typeof struct intset {
    // 编码方式
    unit32_t encoding;
    // 集合包含的元素数量
    unit32_t lenght;
    // 保存元素的数组，保存的数值类型取决于redisObject的encoding属性的值
    int8_t contents[];
} intset;
```

contents数组可以存放的元素类型为：

* INTSET_ENC_INT16
* INTSET_ENC_INT32
* INTSET_ENC_INT64

16,32,64编码对应能存放的数字范围是不一样的。16最少，64最大。若希望存放的数值更大，则需要编码升级，其中步骤为：

（1）根据新元素类型来扩展整数集合底层数组空间

（2）将底层数组现有的全部元素都转换成与新元素相同的类型，并将类型转换后的元素放到正确的位上，同样也要维持有序。

（3）将新元素添加到底层数组。

>不支持降级操作



**命令示例**

```
SADD bools kava
SISMEMBER books java   查询是否存在某个value
SCARD books 获取长度
SPOP books 随机返回一个
```



### 字典

redis的字典相当于Java的HashMap，同样也采用了数组+链表解决哈希冲突。

redis使用dictEntry表示节点（键值对)

```
typedef struct dictEntry {
        void *key;
        //值
        union {
            void *value;
            uint64_tu64;
            int64_ts64;
        }v;    
        //指向下个哈希节点，组成链表。
        //当产生哈希冲突时，新节点放在表头
        struct dictEntry *next;
    }dictEntry;
```


dictht结构来定义哈希表

```
typedef struct dictht{
        //哈希表数组
        dictEntry **table;  

        //哈希表大小
        unsigned long size;    

        //哈希表大小掩码，用于计算索引值
        unsigned long sizemark;     

        //哈希表已有节点数量
        unsigned long used;

    }dictht
```
此外redis还再封装了一层，即外部的字典：

```
typedef struct dict {
    //指向dictType结果的指针，它实现了一些操作键值对的函数
    dictType *type;
    //私有数据，这是需要传给上面特定函数的可选参数
    void *privdata;
    //两张哈希表，ht[0]是原生哈希表，ht[1]是用于扩容的哈希表
    dictht ht[2];
    //当rehash不进行时，值为-1
    int rehashidx;  

}dict;

typedef struct dictType{
    //计算哈希值
    unsigned int (*hashFunction)(const void * key);
    //复制键
    void *(*keyDup)(void *private, const void *key);
    //复制值
    void *(*valDup)(void *private, const void *obj);  
    //对比键
    int (*keyCompare)(void *privdata , const void *key1, const void *key2)
    //销毁键
    void (*keyDestructor)(void *private, void *key);
    //销毁值
    void (*valDestructor)(void *private, void *obj);  
}dictType
```

大致结构如下：

![](https://user-gold-cdn.xitu.io/2020/6/15/172b6686690aff04?w=891&h=489&f=jpeg&s=114298)

#### rehash过程

正常情况下，当hash表中元素个数等于第一维数组的长度时，就会开始扩容，扩容的新数组是原来的2倍。此外如果hash表因元素逐渐删除而变得稀疏，那么当元素个数低于数组长度的10%时，redis会开始缩容。

在对哈希表进行扩展或者收缩操作时，reash过程并不是一次性地完成的，而是渐进式地完成的，这是为了防止数据量过大导致服务器停止服务。

rehash具体过程如下：


（1）将字典的属性rehashidx设置为0，表示rehash开始，它会记录某一时刻rehash的下标。

（2）在rehash期间，它会将ht[0]中rehashidx索引对应的值rehash到ht[1]。此时也可以对字典进行增删改查操作，它会在ht[0]和ht[1]完成操作，比如查询操作是在两个数组里完成；添加操作是只在ht[1]完成。

（3）所有键值对完成rehash后，将rehashidx设置为-1，表示rehash完成。

示例过程可参照下图：

![](https://user-gold-cdn.xitu.io/2020/6/15/172b676a5e90c2da?w=765&h=3606&f=png&s=406495)

**命令示例**

```
HSET books java "think in java"  
HGETALL books  											#key和value是换行出现的
HGET books java
HMSET books java "effetive java" puthon "learn python"  #批量设置
```



### 有序列表(zset)

zset类似Java中SortedSet和HashMap的结合体，即它存储的value是唯一性的，另一方面每个value都有一个代表排序的权重score值。它的内部实现是用跳跃表数据结构。

**命令示例**

```
ZADD books 9.0 "think in java" 
ZRANGE books 0 -1  				#按权值从小到大列出value值
ZREVRANGE books 0 -1  			#从大到小列出
ZCARD books 					#返回books数量
ZSCore books "think in java" 	#获取指定value的权值，可能存在小数点精度问题
ZREM books "think in java"  	#删除value
```



#### 跳跃表

**什么是跳表**

对于一个**有序**链表，如果希望查找某一个数，则需要从头到尾去匹配。为了提高搜索速度，我们可以在原有的链表上，再加一个链表


![](https://user-gold-cdn.xitu.io/2020/6/15/172b6efc1e557e88?w=861&h=74&f=png&s=7698)

此处新链表是由上一层链表的偶数个节点构成，当我们执行查找时，先在最高层链表进行查找，若没有找到则到下一层去查找。例如此时如果我们希望查找节点17，搜寻顺序是6->9->17，发现下一节点21比19大，则我们跑到下一层查找，即可找到19。

上面的例子中上层节点数与下一层节点数比例是1：2，但实际使用中，链表是要经过多次添加/删除的，因此跳表（skip list）不强制要求 1:2，一个节点要不要被索引，建几层的索引，都在节点插入时由抛硬币决定。虽然索引的节点、索引的层数是随机的，但为了保证搜索的效率，要大致保证每层的节点数目与上节的结构相当。

**redis的跳跃表** 

redis的跳跃表实现由zskiplist和zskiplistNode两个结构组成。其中zskiplistNode则表示跳跃表的节点，zskiplist保存跳跃表的信息(表头，表尾节点，长度)。

```
typeof struct zskiplistNode {
        // 后退指针
        struct zskiplistNode *backward;
        // 分值
        double score;
        // 成员对象
        robj *obj;
        // 层
        struct zskiplistLevel {
                // 前进指针
                struct zskiplistNode *forward;
                // 跨度
                unsigned int span;
        } level[];
} zskiplistNode;
```

跳表节点结构大致为：

![](https://user-gold-cdn.xitu.io/2020/6/15/172b6fbb5e15767d?w=760&h=352&f=png&s=59395)

跳表结构

```
typeof struct zskiplist {
        // 表头节点，表尾节点
        struct skiplistNode *header,*tail;
        // 表中节点数量
        unsigned long length;
        // 表中最大层数
        int level;
} zskiplist;
```

整个跳表示例图如下：

![](https://user-gold-cdn.xitu.io/2020/6/15/172b6fd5281396c5?w=767&h=389&f=png&s=98960)



## Java使用Redis

在安装完Redis后，首先打开本地Redis服务

```
redis-server.exe
```

然后是java程序：

```java
        Jedis jedis = new Jedis("localhost");
        System.out.println("服务正在运行：" + jedis.ping());	//输出PONG
        //字符串
        jedis.set("mstr", "mval");
        System.out.println(jedis.get("mstr"));
        //列表
        jedis.lpush("mlist", "3ejy");
        jedis.lpush("mlist", "C2y");
        jedis.lpush("mlist", "on1");
        List<String> list = jedis.lrange("mlist", 0, 2);
        for(int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
```



## 过期删除

我们可以使用 Expire 命令来设置 key 的过期时间（单位是秒）

```
> SET db redis
OK
> EXPIRE db 60
(integer) 1
```



### 删除策略

首先介绍两种删除策略：

（1）主动删除策略：

- **定时删除**：在设置键的过期时间的同时，创建一个定时器（timer），让定时器在键的过期时间来临时，立即执行对键的删除操作。这种方式对内存友好，但浪费CPU资源
- **定期删除**：每隔一段时间，程序对数据库进行一次检查，删除里面的过期键。至于要删除多少过期键，以及要检查多少个数据库，则由算法决定。

（2）被动删除策略

- **惰性删除策略**：放任键过期不管，但是每次从键空间中获取键时，都检查取得的键是否过期，如果过期的话，就删除该键；如果没有过期，就返回该键。这种方式对CPU友好，但浪费内存资源

Redis服务器实际使用的是惰性删除和定期删除两种策略：

（1）Redis的惰性删除策略由 db.c/expireIfNeeded 函数实现，所有键读写命令执行之前都会调用 expireIfNeeded 函数对其进行检查，如果过期，则删除该键，然后执行键不存在的操作；未过期则不作操作，继续执行原有的命令。

（2）Redis的定期删除策略由redis.c/activeExpireCycle 函数实现，函数以一定的频率运行，每次运行时，都从一定数量的数据库中取出一定数量的随机键进行检查，并删除其中的过期键。





## 参考资料

[干货 | SQL 与 NoSQL还在傻傻分不清？](https://zhuanlan.zhihu.com/p/63371253)

[【3y】从零单排学Redis【青铜】](https://segmentfault.com/a/1190000016837791)

[跳表──没听过但很犀利的数据结构](https://lotabout.me/2018/skip-list/)