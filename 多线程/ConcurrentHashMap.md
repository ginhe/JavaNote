# 前序

在java.util包下提供了一些线程安全的容器类，如Vector和HashTable。但这些容器是通过sychronized实现实现同步，这样读写均需要锁操作，导致性能低下。Java提供了一些代替同步容器的并发容器，使用这些容器可以提高并发访问性。

![img](https://img-blog.csdnimg.cn/20200429181207339.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



# ConcurrentMap接口

ConcurrentMap接口继承了Map接口，在Map接口的基础上又定义了4个方法：

```java
public interface ConcurrentMap<K, V> extends Map<K, V> {
    
    //插入元素。如果插入的key值相同，不会替换value值
    V putIfAbsent(K key, V value);

    //移除元素。该方法增加了对value的判断，如果要删除的key-value不能与Map中原有的key-value对应上，则不会删除该元素;
    boolean remove(Object key, Object value);

    //替换元素。该方法增加了对value的判断，只有key-value能与Map中原有的key-value对应上，才进行替换操作;
    boolean replace(K key, V oldValue, V newValue);

   //与上面方法不同的是：该方法不会对Map中原有的key-value进行比较，若key存在则直接替换
    V replace(K key, V value);

}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



# ConcurrentHashMap类

**线程不安全的HashMap**

在并发编程中使用HashMap可能导致程序死循环，这是因为多线程会导致HashMap的Entry链表形成环形数据结构，即Entry的next节点永不为空，进而产生死循环获取Entry。

**效率低下的HashTable**

HashTable容器使用synchronized来保证线程安全，当一个线程访问HashTable的同步方法，其他线程会被进入阻塞或轮询状态，如线程1使用put进行元素添加，线程2不但不能使用put方 法添加元素，也不能使用get方法来获取元素，所以竞争越激烈效率越低。

**ConcurrentHashMap的锁分段技术**

HashTable效率低下的原因是所有访问HashTable的 线程都必须竞争同一把锁，而**ConcurrentHashMap**的锁分段技术将数据分成一段一段的存储，给每一段数据配一把锁，这样当一个线程占用锁访问其中一个段数据的时候，其他段的数 据也能被其他线程访问。



## **JDK 1.7下的**ConcurrentHashMap

ConcurrentHashMap在JDK 1.7中采用了数组+Segment+分段锁的方式实现，Segment是一个继承自ReentrantLock的锁，Segment数组维护了HashEntry的数组table。HashEntry本质是一个K-V存储结构，内部存储了目标对象的Key和Value，同时HashEntry也是一个链式结构，内部维护了下一个HashEntry的变量next。

<img src="https://img-blog.csdnimg.cn/20200429190203987.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70" alt="img" style="zoom: 80%;" />![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

分段锁机制将数据分段，对每一段数据分配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。对于一些方法需要跨段，比如说size()、isEmpty()、containsValue()，它们可能需要锁定整个表而非某个段，因此需要按顺序锁定所有段，操作完后按顺序释放所有的锁。

**ConcurrentHashMap大致结构**

```java
public class ConcurrentHashMap<K, V> extends AbstractMap<K, V> implements ConcurrentMap<K, V>, Serializable {

        static final int DEFAULT_INITIAL_CAPACITY = 16;
        //默认的负载因子为0.75
        static final float DEFAULT_LOAD_FACTOR = 0.75f;
		//默认数组长度
        static final int DEFAULT_CONCURRENCY_LEVEL = 16;

        final Segment<K,V>[] segments;

   static final class Segment<K,V> extends ReentrantLock implements Serializable {
         transient volatile HashEntry<K,V>[] table;
         ...
    }

    static final class HashEntry<K,V> {
        final int hash;
        final K key;
        volatile V value;
        volatile HashEntry<K,V> next;
    }

}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### ConcurrentHashMap的初始化

```java
public ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel) {
    
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS;
    // Find power-of-two sizes best matching arguments
    int sshift = 0;
    int ssize = 1;
    //设置segments数组长度
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
    this.segmentShift = 32 - sshift;
    this.segmentMask = ssize - 1;
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    //计算每一个segment中table的数量cap
    int c = initialCapacity / ssize;
    if (c * ssize < initialCapacity)
        ++c;
    int cap = MIN_SEGMENT_TABLE_CAPACITY;
    while (cap < c)
        cap <<= 1;
    // create segments and segments[0]
    Segment<K,V> s0 =
            new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                    (HashEntry<K,V>[])new HashEntry[cap]);
    Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
    UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
    this.segments = ss;
}
```

我们将构造函数分为两部分讲解：

**（1）设置segments数组长度**

构造函数中第三个参数concurrencyLevel默认为DEFAULT_CONCURRENCY_LEVEL的值 ，即16。

Segment数组的最终大小ssize**一定是大于或等于concurrentLevel的最小的2的次幂。**

```java
if (concurrencyLevel > MAX_SEGMENTS)
   concurrencyLevel = MAX_SEGMENTS;
    int sshift = 0;
/*
	我们要通过hash % length来计算放在桶的下标
	当segments[].length的值为2次幂次方时，hash % length。正好等于hash & (length-1)
	又由于&要比%运算快，因此我们需要控制segments数组的大小要大于等于concurrencyLevel
    的最小2的n次方数
*/
    int ssize = 1;
        while (ssize < concurrencyLevel) {
              ++sshift;
            ssize <<= 1;
        }
    segmentShift = 32 - sshift;
    // segmentMask = segments[].length - 1
    segmentMask = ssize - 1;
    this.segments = Segment.newArray(ssize);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**（2）初始化每个segment中的HashEntry数组数量**

方法的第一个参数initialCapacity默认为DEFAULT_INITIAL_CAPACITY 的值，即16，它表示HashEntry数组的总共大小，ssize则是segment数组大小。
变量c = initialCapacity / ssize，它表示每个segment的HashEntry数组数量。如果c大于1，就会取大于等于c的2的N次方值，因此cap最小为2，即每个segment最少有两个HashEntry数组

```java
if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
        int c = initialCapacity / ssize;
        if (c * ssize < initialCapacity)
            ++c;
        int cap = 1;
        while (cap < c)
            cap <<= 1;
        for (int i = 0; i < this.segments.length; ++i)
            this.segments[i] = new Segment<K,V>(cap, loadFactor);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### **put操作**

put操作主要分为两步：

（1）定位到放入segment数组的下标j，并确保该位置已被初始化；

（2）调用segment的put方法

```java
public V put(K key, V value) {
        Segment<K,V> s;
        if (value == null)
            throw new NullPointerException();
        int hash = hash(key);
        int j = (hash >>> segmentShift) & segmentMask;
        if ((s = (Segment<K,V>)UNSAFE.getObject          
             (segments, (j << SSHIFT) + SBASE)) == null) 
            s = ensureSegment(j);
        return s.put(key, hash, value, false);
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

然后是segment对象的put方法：

（1）由于put方法需要对共享变量进行写入操作，因此需要加锁。该节点通过tryLock()方法尝试加锁，若不成功表示当前锁已被其他线程持有，则执行`scanAndLockForPut()`方法：在`scanAndLockForPut`方法中，会通过重复执行`tryLock()`方法尝试获取锁，如果执行`tryLock()`方法次数超过上限后，会执行lock()方法挂起当前线程，等待其他线程unlock()。

（2）根据HashEntry[].length - 1 & hash来得到HashEntry数组的下标index，然后放入对应的HashEntry数组里。

（3）判断Segment里的HashEntry数组是否超过阈值（threshold），如果超过，则对数组进行扩容。

```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
            HashEntry<K,V> node = tryLock() ? null :
                scanAndLockForPut(key, hash, value);
            V oldValue;
            try {
                HashEntry<K,V>[] tab = table;
                int index = (tab.length - 1) & hash;
                HashEntry<K,V> first = entryAt(tab, index);
                for (HashEntry<K,V> e = first;;) {
                    if (e != null) {
                        K k;
                        if ((k = e.key) == key ||
                            (e.hash == hash && key.equals(k))) {
                            oldValue = e.value;
                            if (!onlyIfAbsent) {
                                e.value = value;
                                ++modCount;
                            }
                            break;
                        }
                        e = e.next;
                    }
                    else {
                        if (node != null)
                            node.setNext(first);
                        else
                            node = new HashEntry<K,V>(hash, key, value, first);
                        int c = count + 1;
//若c超出阈值threshold，需要扩容并rehash。
                        if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                            rehash(node);
                        else
                            setEntryAt(tab, index, node);
                        ++modCount;
                        count = c;
                        oldValue = null;
                        break;
                    }
                }
            } finally {
                unlock();
            }
            return oldValue;
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### get方法

由于get方法只需要读且涉及到的共享变量都使用volatile修饰，因此无需加锁。

```java
public V get(Object key) {
        Segment<K,V> s; 
        HashEntry<K,V>[] tab;
        int h = hash(key);
        long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
        //先定位Segment，再定位HashEntry
        if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
            (tab = s.table) != null) {
            for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
                     (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
                 e != null; e = e.next) {
                K k;
                if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                    return e.value;
            }
        }
        return null;
    }
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



## JDK 1.8

JDK1.8的ConcurrentHashMap采用的节点Node数组+链表/红黑树+CAS+synchronized来保证并发安全，不允许null作为key和value。

![](https://user-gold-cdn.xitu.io/2020/7/20/17369d15e01d32a5?w=782&h=319&f=png&s=94502)

synchronized只锁定当前链表或红黑二叉树的首节点，这样只要hash不冲突，就不会产生并发，效率又提升N倍。

ConcurrentHashMap的大致结构如下。

（1）Node类包装了key-value键值对，所有插入ConcurrentHashMap的数据都包装在这里面。它不允许调用setValue方法直接改变Node的value域。

（2）当链表长度过长的时候，会转换为TreeNode。与HashMap不相同的是，它并不是直接转换为红黑树，而是把这些结点包装成TreeNode放在TreeBin对象中，由TreeBin完成对红黑树的包装。TreeNode继承自Node类。

（3）TreeBin类负责包装很多的TreeNode节点，在实际的ConcurrentHashMap“数组”中，存放的是TreeBin对象，而不是TreeNode对象。

（4）ForwardingNode类用于连接两个table，它包含一个nextTable指针，用于指向下一张表。

### 属性

```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V> implements ConcurrentMap<K,V>, Serializable {
    
    static final int MOVED     = -1; //  hash值是-1，表示这是一个forwardNode节点
    static final int TREEBIN   = -2; //  hash值是-2  表示这时一个红黑树节点

//当插入新数据put()或则删除数据remove()时，会通过addCount()方法更新baseCount
    private transient volatile long baseCount; 

    private transient volatile CounterCell[] counterCells;
    //节点数组
    transient volatile Node<K,V>[] table;
    //扩容时，将table中的元素迁移至nextTable . 扩容时非空
    transient volatile Node<K,V>[] nextTable

    /**
     * 控制标志符
     * 负数: 代表正在进行初始化或扩容操作，其中-1表示正在初始化，-N 表示有N-1个线程正在进行扩容操作
     * 正数或0: 代表hash表还没有被初始化，这个数值表示初始化或下一次进行扩容的大小，类似于扩容阈值
     * 它的值始终是当前ConcurrentHashMap容量的0.75倍（加载因子）。
     * 当实际容量 >= sizeCtl，则扩容
     */
    private transient volatile int sizeCtl;
    ...
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;
        ...
    }
	
    static final class TreeNode<K,V> extends Node<K,V> {
        TreeNode<K,V> parent;
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;   
        boolean red;
       ...
    }
    //TreeBin包装了TreeNode类。桶table实际存放的是TreeBin而非TreeNode
    static final class TreeBin<K,V> extends Node<K,V> {
        TreeNode<K,V> root;
        volatile TreeNode<K,V> first;
        volatile Thread waiter;
        volatile int lockState;
 
        static final int WRITER = 1; // set while holding write lock
        static final int WAITER = 2; // set when waiting for write lock
        static final int READER = 4; // increment value for setting read lock
        ...
    }
    static final class ForwardingNode<K,V> extends Node<K,V> {
        final Node<K,V>[] nextTable;
        ...
    }
}
    
```

### 构造方法

ConcurrentHashMap的构造方法仅仅是设置了一些参数。

```java
public ConcurrentHashMap() {}

public ConcurrentHashMap(int initialCapacity) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException();
    	//tableSizeFor方法返回大于方法参数initialCapacity的最小2幂次方整数
        int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
                   MAXIMUM_CAPACITY :
                   tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
        this.sizeCtl = cap;
}

// 构造一个空的 Map 映射，并给定其初始容量与加载因子
public ConcurrentHashMap(int initialCapacity, float loadFactor) {
    this(initialCapacity, loadFactor, 1);
}

// 构造一个空的 Map 映射，并给定其初始容量，加载因子与预估的并发更新的线程数
public ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel) {
    
    if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (initialCapacity < concurrencyLevel)   // Use at least as many bins
        // 该情况下，一个更新线程负责一个HashEntry
        initialCapacity = concurrencyLevel;   // as estimated threads
    // 确定 table 的真实长度 = 桶长度 initialCapacity / 负载因子 loadFactor
    // 比如要存30个元素，构造Map的时候传入30和0.75，那么table真实容量就应该是 30/0.75。保证你要存的元素数量是table容器的0.75倍
    long size = (long)(1.0 + (long)initialCapacity / loadFactor);
    int cap = (size >= (long)MAXIMUM_CAPACITY) ?
        MAXIMUM_CAPACITY : tableSizeFor((int)size);
    this.sizeCtl = cap;
}

private static final int tableSizeFor(int c) {
    int n = c - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}

```

### 初始化方法initTable

整个table的初始化是在向ConcurrentHashMap中插入元素时发生的。如调用put、computeIfAbsent、compute、merge等方法的时候。当它们发现桶table为空或长度为0，则调用initTable方法初始化

```java
private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {
//sizeCtl为负数表示有其他线程正在进行初始化或扩容操作，于是把当前线程挂起。
            if ((sc = sizeCtl) < 0)
                Thread.yield();
//利用CAS方法把sizectl的值置为-1 表示本线程正在进行初始化
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    //当sc大于0时，表示的是初始化或下一次进行扩容的大小，否则使用默认长度16
                    if ((tab = table) == null || tab.length == 0) {
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
                        //计算数组的可用大小=实际大小 * 加载因子0.75
                        sc = n - (n >>> 2); //无符号右移两位 n-(1/4)n
                    }
                } finally {
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
```



### 扩容方法 transfer

**什么时候扩容**

当往hashMap中成功插入一个key-value节点时，有可能触发扩容动作：

（1）桶中某个链表长度达到达到8，但桶长度小于64

（2）新增元素后，元素个数达到扩容阈值触发扩容。

（3） 调用 putAll 方法，发现容量不足以容纳所有元素时候触发扩容。

扩容操作分为两步骤：

1. 构建一个容量是原来两倍的nextTable，此步骤是单线程完成的；
2. 将原来table中的元素复制到nextTable中，此步骤允许多线程进行操作。

```java
    private final void transfer(Node[] tab, Node[] nextTab) {
        int n = tab.length, stride;
        //每个线程处理的桶数量stride = 桶长度n 除以 8后再除以CPU核心数NCPU。如果得到的结果小于16，则使用16。
        //目的是让每个 CPU 处理的桶一样多，避免出现转移任务不均匀的现象，如果桶较少的话，默认一个 CPU（一个线程）处理 16 个桶
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; 
        //如果扩容桶为空
        if (nextTab == null) {            
            try {
                @SuppressWarnings("unchecked")
                // 创建node数组，容量为当前的两倍
                Node[] nt = (Node[])new Node[n << 1];
                nextTab = nt;		//赋值给扩容桶
            } catch (Throwable ex) {     
                // 若扩容时出现OOM异常，则将阈值设为最大，表明不支持扩容
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            //更新转移下标为旧桶的length
            transferIndex = n;
        }
        int nextn = nextTab.length;
        // 创建一个 fwd 节点用于占位：当别的线程发现这个槽位中是 fwd 类型的节点，则跳过这个节点。
        ForwardingNode fwd = new ForwardingNode(nextTab);
        //advance为true表示需要再次推进一个下标，反之则不能推进，需要将当前的下标处理完毕才能继续推进
        boolean advance = true;
        // 完成状态，如果是 true，就结束此方法。
        boolean finishing = false; 
        //i为当前线程可以处理的当前区间的最大下标，boune表示当前线程可以处理的当前桶区间最小下标  [boune, i]
        for (int i = 0, bound = 0;;) {
            Node f; int fh;
            while (advance) {
                int nextIndex, nextBound;
            // （1）如果对 i 减一大于等于 bound（表示该区间还需要继续做任务），或者任务完成了，则修改推进状态为 false表示不能推进了。
                if (--i >= bound || finishing)
                    advance = false;
           // （2）通常第一次进入循环，上面的判断无法通过，从而走下面的 nextIndex 赋值操作（获取最新的转移下标：旧桶length）。
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;		// 如果小于等于0，说明没有桶区间了，i 改成 -1，推进状态变成 false，表示扩容结束了，当前线程可以退出了
                    advance = false;
                }
          //（3）通过CAS设置transferIndex属性值 = 旧桶.length - 区间值stride（每个线程处理的桶长度）
                else if (U.compareAndSwapInt(
                    		this, TRANSFERINDEX, nextIndex,nextBound = (nextIndex > stride ?nextIndex - stride : 0))
                         ) {	//要交换的值TRANSFERINDEX，期望值nextIndex，更新后的值为nextIndex - stride（或者0）
                    
                    bound = nextBound;	//当前线程可以处理的最小当前区间最小下标
                    i = nextIndex - 1;	//当前线程可以处理的当前区间的最大下标
                    advance = false;	//防止在没有成功处理一个桶的情况下却进行了推进，导致漏掉某个桶。
                }
            }
            // 如果i小于0，也就是上面代码中的（2），则说明领取最后一段区间的线程扩容结束
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                //如果所有的节点都已经完成复制工作  就把nextTable赋值给table 清空临时对象nextTable
                if (finishing) {	
                    nextTable = null;
                    table = nextTab;
                    //设置新扩容阈值
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                //否则利用CAS方法更新扩容阈值，在这里面sizectl值减一，说明新加入一个线程参与到扩容操作
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    //如果sc-2不等于标识符左移 16 位不，则说明扩容没有结束
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;	//当前线程结束方法
                    finishing = advance = true;	// 如果相等，扩容结束了，更新 finising 变量
                    i = n; 		// 再次循环检查表
                }
            }
            else if ((f = tabAt(tab, i)) == null)	// 获取旧桶i 下标位置的变量，如果是 null，就使用 fwd 占位。
                advance = casTabAt(tab, i, null, fwd);
            else if ((fh = f.hash) == MOVED)	// 如果不是 null 且 hash 值是 MOVED，说明别的线程已处理过该节点
                advance = true; // 再次推进一个下标
            else {	//如果到了此处说。防止 putVal 的时候向链表插入数据
                synchronized (f) {
                    //确保f是i位置上桶的节点
                    if (tabAt(tab, i) == f) {
                        Node ln, hn;
                        //如果当前桶是链式结构
                        if (fh >= 0) {
                            //对旧桶长度进行与运算。
                            int runBit = fh & n;
                            Node lastRun = f;
                            //类似于1.8HashMap，只需要看新增的1bit是0还是1进行分类
                            for (Node p = f.next; p != null; p = p.next) {
                                //n是就数组长度，不是长度-1
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node(ph, pk, pv, ln);
                                else
                                    hn = new Node(ph, pk, pv, hn);
                            }
                            //在nextTable的i位置上插入一个链表
                            setTabAt(nextTab, i, ln);
                            //在nextTable的i+n的位置上插入另一个链表
                            setTabAt(nextTab, i + n, hn);
                            //在table的i位置上插入forwardNode节点  表示已经处理过该节点
                            setTabAt(tab, i, fwd);
                            //设置advance为true 返回到上面的while循环中 就可以执行i--操作
                            advance = true;
                        }
                        //当前桶是红黑树结构，操作和上面的类似
                        else if (f instanceof TreeBin) {
                            TreeBin t = (TreeBin)f;
                            TreeNode lo = null, loTail = null;
                            TreeNode hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode p = new TreeNode
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            //如果扩容后已经不再需要tree的结构 反向转换为链表结构
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                    }
                }
            }
        }
    }
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### get方法

大致流程为：

（1）计算key的哈希值h

（2）通过h & table[].length -1确定桶下标对应的节点e。如果e不为空且e的key值和hash值是否和h相等，若是则返回

（3）如果e是树节点，则从树里找到对应的值（或空）

（4）否则是一个链表节点，遍历链表找到值

```java
    public V get(Object key) {
        Node[] tab; Node e, p; int n, eh; K ek;
        //（1）计算hash值
        int h = spread(key.hashCode());
        //（2）根据key.hashCode & table[].length - 1来确定节点e，
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
            //（2）桶首节点的key与查找的key相同，再判断一下key是否相同
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;		
            }
            //（3）如果节点e的哈希值eh小于0，表示它是一个树节点
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            //（4）否则是一个链表节点
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }

static final int HASH_BITS = 0x7fffffff;
static final int spread(int h) {
    return (h ^ (h >>> 16)) & HASH_BITS;
}
```



### put方法

大致流程为：

（1）由于ConcurrentHashMap不允许key或value为null，因此首先判断key和value是否为null。

（2）重新计算hash值，然后判断当前table是否为空，若为空则初始化table。通过table.length - 1 & hash 得到位置i。通过tabAt方法获取对应位置节点f，如果f为空，则通过cas操作插入新Node节点，然后退出方法。

（3）如果f是占位节点，表明有其它线程正在扩容，则一起进行扩容操作；

（4）将f节点作为锁对象，并再次判断位置i的头节点是否为f。

​	（4-1）如果f是链表节点，则根据头节点f遍历链表：

​		（4-1-1）如果找到hash值与key值相同的节点且方法参数onlyIfAbsent为false，则替换旧值；

​		（4-1-2）若链表中找不到，在链表尾部插入该节点

​	（4-2）若节点f是树节点，则遍历红黑树

（5）若链表长度超过默认值8，将链表转为红黑树然后节点数量+1，校验是否超过阈值，若超过则扩容。

```javascript
  public V put(K key, V value) {
        return putVal(key, value, false);
    }

    final V putVal(K key, V value, boolean onlyIfAbsent) {
//（1）ConcurrentHashMap不允许key或value为null
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            //（2）如果桶为空桶长度为0，则初始化得到桶
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
//（2）若table.length - 1 & hash得出的位置i上的节点f为null，则CAS插入新Node节点。
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   
            }
//        （3）如果位置i的节点f是哈希值是MOVED，表示它是占位节点
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);	//帮助扩容
            else {
                V oldVal = null;
//        （4）锁住当前位置i的节点f（头节点）
                synchronized (f) {
    //       （4） 为防止之前被其他线程修改，需要判断节点f是否为数组下标i的节点。
                    if (tabAt(tab, i) == f) {
        //        （4-1）如果当前节点是链表节点
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {	//（4-1）根据头节点遍历链表
                                K ek;
        		//	（4-1-1）若hash值与key值相同且onlyIfAbsent为false，则替换旧值
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                 //	（4-1-2）若链表中找不到，在链表尾部插入该节点
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                    //（4-2）若节点f是树节点，则遍历红黑树
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                 //（5）若链表长度超过默认值8，将链表转为红黑树
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
//（5）节点数+1，若超过阈值则扩容
        addCount(1L, binCount);
        return null;
    }
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### size方法

在多线程环境下，ConcurrentHashMap的table数量是不确定的，因此该方法返回的是个估计值。

其中元数个数保存在baseCount，部分元素的变化个数保存在CounterCell数组counterCells中，通过累加baseCount和CounterCell数组中的数量，即可得到元素的总个数；

```java
    public int size() {
        long n = sumCount();
        return ((n < 0L) ? 0 :
                (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :
                (int)n);
    }
    
    final long sumCount() {
        CounterCell[] as = counterCells; CounterCell a;
        long sum = baseCount;
        if (as != null) {
            for (int i = 0; i < as.length; ++i) {
                if ((a = as[i]) != null)
                    sum += a.value;
            }
        }
        return sum;
    }
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)





# 参考资料

[ConcurrentHashMap JDK1.8](https://www.cnblogs.com/duanxz/archive/2012/10/08/2714933.html)
[ConcurrentHashMap JDK1.8](https://juejin.im/post/5b53d1adf265da0f70070e3d#heading-0)

[ConcurrentHashMap JDK1.8的扩容方法](https://juejin.im/post/5b00160151882565bd2582e0)