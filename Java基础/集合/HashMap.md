# 概述
HashMap用于存储键值对，它的大致继承结构如下：


![](https://user-gold-cdn.xitu.io/2020/5/4/171de39aceadf83d?w=1060&h=419&f=png&s=169287)


```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
}
```

# 底层数据结构
## JDK1.8之前
在JDK1.8之前，HashMap底层由数组和链表组成，数组的长度有限，因此使用链表来解决哈希冲突。

![](https://user-gold-cdn.xitu.io/2020/5/4/171de3e4ddfb7c2b?w=707&h=379&f=png&s=77917)

存入数组里的每个元素都是一个节点，或者说是一个键值对类型Entry。

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    
    //默认初始化容量初始化=16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 
    //最大容量2^30
    static final int MAXIMUM_CAPACITY = 1 << 30;
    
    //默认加载因子。HashMap的扩容临界值是当前HashMap大小 * 加载因子
    //默认为DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR = 0.75 * 16
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    
    //扩容阔值，当size大于其值会扩容
    //一般情况下threshold=capacity*loadFactor
    int threshold;
    
    //加载因子
    final float loadFactor;
    //数组长度必须是2的n次方。JDK1.8版本同样也是
    transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
    
    static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;
        int hash;
    }
}
```

**为什么负载因子是0.75**

首先我们要明白，我们的key-value作为节点存储在桶table里，当发生Hash冲突时，就会在对应的下标中生成一个链表。当链表长度大于TREEIFY_THRESHOLD时，链表会被转为红黑树。

0.75是一个时间和空间的权衡：如果负载因子是1.0，那么当节点个数为16时才扩容，由于Hash冲突是不可避免的，此是可能出现大量的Hash冲突，底层的红黑树会很复杂，不利于查询效率；而如果查询因子是0.5，那么节点元素达到一半就扩容，Hash冲突也会减少，链表长度或者是红黑树的高度就会降低。查询效率就会增加。但是空间利用率就会降低。

理想状态下，在随机哈希值的情况，对于loadfactor = 0.75 ，虽然由于粒度调整会产生较大的方差，桶中的Node的分布频率服从参数为0.5的**泊松分布。** 



### 如何确定元素在数组（桶）的位置

**计算放入的下标**

桶的长度是有限的，而计算出来的哈希值往往会超出其长度，因此我们需要将哈希值转换为在数组长度范围内，即hash % table.length。

但除法运算要比 与运算&慢得多，因此我们用&代替%，当**table数组的长度为2的n次方时**，table.length - 1是一个后低(n-1)位都为1的二进制，以数组初始长度16为例，16-1=15，与计算出的哈希值做 “与运算”：
>10100101 11000100 00100101         //一个假定的hashcode
>
>00000000 00000000 00001111
>
>-------------------------
>0000000000000000000000101      //  只保留哈希值的末4位



hashCode % table.length 等价于 hashCode & (table.length - 1)。




### 扰动操作

前面我们说的哈希值并不是指key的hashCode值，我们还需要对hashCode值做扰动操作，避免发生哈希冲突。

举例来说，key1的hashCode是11280384，用16进制表示是0x0AC20000;
key2的hash是1568768，用16进制表示是0x017F0000。数组长度默认为16。如果不进行扰动，直接用hashCode计算下标：


* key1计算的下标：0AC20000 & 0000000F = 0
* key2计算的下标：017F0000 & 0000000F = 0

这样就产生了冲突，原因是这两个key的hashCode虽然完全不同，但低位相同，高位不同，而HashMap在计算数组下标时，如果table.length较小，则只会取低位，高位完全失效了。


```java
final int hash(Object k) {
    int h = hashSeed;
    if (0 != h && k instanceof String) {
        return sun.misc.Hashing.stringHash32((String) k);
    }
    //扰动操作
    h ^= k.hashCode();
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```
加入扰动操作后，同样的例子：

0AC20000 >>> 20 = 000000AC
>0000 1010 1100 0010 0000 0000 0000 0000 右移20位后
>变成
>
>0000 0000 0000 0000 0000 0000 1010 1100

0AC20000 >>> 12 = 0000AC20
>0000 1010 1100 0010 0000 0000 0000 0000 右移12位后
>变成
>
>0000 0000 0000 0000 1010 1100  0010 0000

然后000000AC ^ 0000AC20  = 0000AC8C
>0000 0000 0000 0000 0000 0000 1010 1100
>
>0000 0000 0000 0000 1010 1100  0010 0000
>
>---------------------------------------
>
>0000 0000 0000 0000 1010 1100 1000 1100

接着0AC20000 ^ 0000AC8C = 0AC2AC8C
>0000 1010 1100 0010 0000 0000 0000 0000
>
>0000 0000 0000 0000 1010 1100 1000 1100
>
>---------------------------------------
>0000 1010 1100 0010 1010 1100 1000 1100

这一段代码主要是把hashcode的高低位混在一起，让高位和低位同时发生变化。

然后又进行了另一段的扰动计算：

0AC2AC8C >>> 7 = 00158559
0AC2AC8C >>> 4 = 00AC2AC8
0AC2AC8C ^ 00158559 ^ 00AC2AC8 = 0A7B031D

这样key1扰动完成后的hash是0A7B031D，放入下标为13；而key2的hash值是016A18B6，放入下标是6，没有产生hash冲突。


## JDK1.8及其之后
当链表长度大于阔值（默认为8）时，将链表转为红黑树，以减少搜索时间。

![](https://user-gold-cdn.xitu.io/2020/5/4/171dec6928575c95?w=400&h=372&f=png&s=16377)

**类的属性**


```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    // 桶默认的初始容量是16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;   
    // 桶最大容量2^30
    static final int MAXIMUM_CAPACITY = 1 << 30; 
    // 默认负载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 当桶的结点数大于该值时会转成红黑树
    static final int TREEIFY_THRESHOLD = 8; 
    // 当桶的结点数小于该值时树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 桶中结构转化为红黑树对应的table的最小大小
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 存储元素的数组，下文称为桶。其长度总是2的幂次倍
    transient Node<k,v>[] table; 
    // 存放具体元素的集
    transient Set<map.entry<k,v>> entrySet;
    // 存放元素的个数
    transient int size;
    // 每次扩容和更改map结构的计数器
    transient int modCount;   
    // 临界值 当实际大小(容量*负载因子)超过临界值时，会进行扩容
    int threshold;
    // 负载因子
    final float loadFactor;
    
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  
        TreeNode<K,V> left;    
        TreeNode<K,V> right;   
        TreeNode<K,V> prev;    
        boolean red;           // 判断颜色
    }
}
```
### 如何确定元素在数组（桶）的位置
同样也是通过hash & (table.length - 1)来得到数组下标。但JDK1.8的操作简洁了很多，它将hashCode值右移16位，然后和原hashCode值 “异或运算”得到哈希值。
```java
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```


![](https://user-gold-cdn.xitu.io/2020/5/4/171dedcb708c761a?w=760&h=470&f=png&s=127457)


# 构造方法

```java
// 默认构造函数。
    public HashMap() {		//填充因子0.75f
        this.loadFactor = DEFAULT_LOAD_FACTOR; 
     }
     
     // 包含另一个“Map”的构造函数
     public HashMap(Map<? extends K, ? extends V> m) {
         this.loadFactor = DEFAULT_LOAD_FACTOR;
         //putMapEntries方法将传入的m添加到本map实例中
         putMapEntries(m, false);
     }
     
     // 指定“容量大小”的构造函数
     public HashMap(int initialCapacity) {
         this(initialCapacity, DEFAULT_LOAD_FACTOR);
     }
     
     // 指定“容量大小”和“加载因子”的构造函数
     public HashMap(int initialCapacity, float loadFactor) {
         if (initialCapacity < 0)
             throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
         if (initialCapacity > MAXIMUM_CAPACITY)
             initialCapacity = MAXIMUM_CAPACITY;
         if (loadFactor <= 0 || Float.isNaN(loadFactor))
             throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
         this.loadFactor = loadFactor;
         //计算大于initialCapacity的最小二次幂
         this.threshold = tableSizeFor(initialCapacity);
     }
```

**putMapEntries方法：**

该方法负责根据传入的哈希表m来初始化本map实例：

- 如果本map的桶还未初始化，则先计算m的实际长度t= m.size() / loadFactor + 1
  - 判断t是否大于本map的最大容量：如果是则重新计算本map的临界值threshold = tableSizeFor(t)
- 否则如果本map的桶已初始化且m的元素个数大于本map的临界值，则扩容
- 然后将m里的元素放入本map里。



```java
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        // (1)如果桶还未初始化，则其实际容量为：传入的哈希表里元素个数s / 0/75 + 1.
        if (table == null) { 
            float ft = ((float)s / loadFactor) + 1.0F;
            //(1-1)判断刚刚计算的ft是否小于最大容量MAXIMUM_CAPACITY 
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                    (int)ft : MAXIMUM_CAPACITY);
           
           //(1-1)如果计算出的实际大小ft大于临界值threshhold，那么重新计算临界值threshhold
            if (t > threshold)
                threshold = tableSizeFor(t);			//大于ft的最小二次幂
        }
        //（2）若table已初始化，且传入哈希表的元素个数大于阈值，进行扩容处理
        else if (s > threshold)
            resize();
        // （3）将传入的哈希表中的所有元素添加至HashMap中
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

# put方法

该方法负责将key-value放入hashMap里：

- 如果桶为空或长度为0，则扩容。通过hash & (table.length - 1)计算要放入桶的下标i：

  - 如果位置i为空，则新建节点e并放进桶里

  - 否则表明产生了hash冲突。比较在该位置的节点p的hash值与key值是否和要放进的key值（key通过==和equals方法比较）和hash值相同：若是则e = p

    - 如果不相同的话，如果p是树节点，则将key-value添加进红黑树

      - 如果p是链表节点，则遍历链表：如果存在一个节点的hash值和key值与要放进的相同，则更新旧值。否则将新节点放到链表尾部。
- 最后判断节点个数size是否大于临界值，若是则resize


​      


```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
//变量onlyIfAbsent表示是否替换已存在的key-value。fale表示替换
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    
    Node<K,V>[] tab;  Node<K,V> p;  int n, i;
    //(1)如果桶为空或桶长度为0，调用resize方法进行扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
        
    //（1-1）通过hash & (table.length - 1)计算出桶下标i，若该位置为空，
    //则新建节点并放入其中
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);

    else {  //（1-2）该位置已存在节点，产生hash冲突
        Node<K,V> e; K k;
        //比较第一个节点的hash值和key值与待插入元素的对应值是否相等
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;		//e指向该节点
        //若该节点为红黑树结点
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value); //加入红黑树
        // 如果该节点为链表结点，判断链表中是否存在某一节点的hash值和key值 与目标节点的对应值相等，
        //若存在，则更新其旧值。否则将目标节点插入链表尾部
        else {  
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {	//如果遍历到链表尾节点
                    //则在尾部插入新的结点
                    p.next = newNode(hash, key, value, null);
                    //如果链表长度binCount大于8，就会转变为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) 
                        treeifyBin(tab, hash);
                    break;
                }
                //找到了key和hash值相同的节点
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        //存在某一节点e的hash值和key值与待插入元素相等，更新旧值
        if (e != null) { 
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value; //用传入的参数value更新旧的value值
            afterNodeAccess(e);
            return oldValue; //返回旧的value值
        }
    }
    ++modCount;
    //容量超出就扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}

```

# get方法

该方法根据key的hash值取出value：

- 通过(table.length - 1) & hash计算得出桶下标i，如果该位置下的key和hash值与目标相同，则返回该节点。
- 如果该位置的节点是红黑树节点，则调用`getTreeNode`方法返回节点
- 如果是链表节点，则在链表中遍历查找。



```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    //计算存放在数组table中的位置
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        //判断在该位置的节点是否与目标相等（key，hash值相等）
        if (first.hash == hash && 
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        //该位置的节点为红黑树根节点或链表头结点
        if ((e = first.next) != null) {
            //如果为红黑树结点
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            //否则在链表中遍历查找
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

# resize方法
当HashMap里的元素个数size大于DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY时，需要扩大数组长度。该方法的实现是使用一个新数组代替旧数组。

看一下JDK1.7下的resize方法：

（1）如果扩容前桶数量已达到最大值MAXIMUM_CAPACITY，则把极限值threshold修改为int的最大值Integer.MAX_VALUE。这样之后都不会扩容了。然后返回

（2）调用`transfer`方法将旧桶的值移到新桶newTable里。

​		（2-1）遍历旧桶里src的每一个节点e：释放节点e，然后根据 hash & (newTable.length-1)计算e在newTable里的位置i，然后添加到对应位置。每次添加的新节点都是链表头节点。

（3）修改极限值为新桶长度newCapacity * loadFactor


```java
 void resize(int newCapacity) {   
      Entry[] oldTable = table;    
      int oldCapacity = oldTable.length;  
      //(1)如果扩容前的数组大小已经达到最大值(2^30)，则修改阈值为int的最大值(2^31-1)，这样以后就不会扩容了
      if (oldCapacity == MAXIMUM_CAPACITY) {  
          threshold = Integer.MAX_VALUE; 
          return;
      }
   
     Entry[] newTable = new Entry[newCapacity]; 
      //（2）将数据转移到新的Entry数组里
     transfer(newTable);                         
     table = newTable;   
     //(3)
     threshold = (int)(newCapacity * loadFactor);//修改阈值
 }
 
  void transfer(Entry[] newTable) {
      Entry[] src = table;                   //src引用旧的Entry数组
      int newCapacity = newTable.length;
      for (int j = 0; j < src.length; j++) { 
          Entry<K,V> e = src[j];        //键值对e引用旧数组里的每个元素    
          if (e != null) {
              src[j] = null;  //释放旧Entry数组的对象引用
              do {
                  Entry<K,V> next = e.next;
                  //（2-1）重新计算每个元素在新数组中的位置i，将元素e放入新数组里
                  //然后节点e的next指针指向对应新数组位置，这样每次添加的
                  //新元素都是链表头结点，即newTable[i]指向的节点。
                 int i = indexFor(e.hash, newCapacity); 
                 e.next = newTable[i]; 
                 newTable[i] = e;      
                e = next;             //访问下一个Entry链上的元素
             } while (e != null);
         }
     }
 } 
 
static int indexFor(int h, int length) {
    return h & (length-1);
}
```

下面是JDK1.8的扩容方法：

（1）如果旧桶的长度oldCap大于0且大于等于最大容量MAXIMUM_CAPACITY，则将极限值threshold设置为Integer.MAX_VALUE，然后返回

（2）新桶长度newCap为旧桶长度oldCap的2倍，若newCap小于最大容量MAXIMUM_CAPACITY且oldCap大于等于默认容量DEFAULT_INITIAL_CAPACITY，则新阔值newThr变为旧阔值的2倍。

（3）否则如果旧阔值oldThr大于0，则新桶长度newCap为oldThr。

（4）否则如果旧桶为空，且旧阔值为0（即此是是无参构造初始化里的resize情况），那么newCap为默认长度16，newThr为默认装载因子 * 默认桶长度。

（5）如果newThr为空，则判断ft = newCap * loadFactor是否小于MAXIMUM_CAPACITY且newCap < MAXIMUM_CAPACITY：若是则newThr = ft。否则newThr = Integer.MAX_VALUE

（6）创建一个长度是newThr的新桶newTab，遍历旧桶里的节点e：

​	（6-1）将e置为空。如果e的下一节点不为空，则通过e.hash & (newCap - 1)计算在新桶里的下标；

​	（6-2）如果e是红黑树节点，则需要对红黑树进行拆分；

​	（6-3）如果e的下一节点非空，则遍历链表进行映射


```java
//当初始化哈希表(table==null) 或当前数组容量过小，就会扩容
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table; //oldTab指向旧的table数组
    //oldTab不为null的话，oldCap为原table的长度
    //oldTab为null的话，oldCap为0
    int oldCap = (oldTab == null) ? 0 : oldTab.length; 
    int oldThr = threshold; //阈值
    int newCap, newThr = 0;
    if (oldCap > 0) {  
        //（1）若原数组容量oldCap大于等于最大容量MAXIMUM_CAPACITY，则
        //将阔值设置为Integer.MAX_VALUE
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE; 
            return oldTab;
        }
        // （2）新容量newCap为原容量oldCap的2倍，若newCap小于MAXIMUM_CAPACITY，
        //且oldCap大于默认值16，则新阔值 newThr也扩大为原来的2倍。
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; 
    }
    /*
        （3）如果HashMap不是调用无参构造初始化的，那么threshhold肯定调用了
        tabSizeFor方法变成2的整数次幂，因此旧阔值作为数组长度。
    */
    else if (oldThr > 0) 
        newCap = oldThr; 
    /*
        （4）否则如果是调用无参构造（table == null,threshhold = 0）初始化的，
        则新容量等于默认容量，新阔值等于默认加载因子*默认初始化容量
    */
    else { 
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // （5）计算新阔值
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor; //新容量 * 加载因子
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    //创建一个长度为新容量newCap的新Node数组
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap]; 
    table = newTab;
    //原来的table不为null
    if (oldTab != null) {
        // （6）把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            //原table中下标j位置不为null
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null; //将原来的table[j]赋为null，及时GC
                if (e.next == null) //	（6-1）如果该位置没有链表
                    //通过新的容量计算在新的table数组中的下标
                    newTab[e.hash & (newCap - 1)] = e; 
                else if (e instanceof TreeNode) 
                    //	（6-2）如果是红黑树结点，重新映射时，需要对红黑树进行拆分
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { //	（6-3）否则遍历链表，重新映射
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 若链表节点放入新桶下标的位置不变（后面会说明原因）
                        if ((e.hash & oldCap) == 0) {
                            //如果loTail为null，即新数组中该位置尚未有节点，因此新添节点是头节点
                            if (loTail == null) 
                                loHead = e;
                            //否则新添节点是尾节点
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        //否则元素在新数组下标为原位置+旧容量，同样也是尾插法。
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 如果新添元素放入新桶下标与原桶下标j相同，则将对应元素
                    //放入对应下标j。否则放入原索引+oldCap下标中，
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead; 
                    }
                    
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}

```
**计算链表节点在新数组的下标**

JDK1.7的扩容方法中，计算链表节点在新数组的下标采用的是重新计算hash值。

而1.8版本不是这么做：由于数组的更新长度为原来的2倍，因此元素的位置要么在原位置，要么在原位置再移动2次幂的位置。
如下图所示，
![](https://user-gold-cdn.xitu.io/2020/5/4/171dfc8dc01fd71e?w=1641&h=446&f=png&s=52883)

图(a)表示原数组中两个key对应的下标，右边表示(n-1) & hash得到的桶下标；图(b)则表示经过扩容后，新桶的新长度为32，新的n-1比原来的n-1在高位多了1bit。也正是这多出的高位1bit，使得key2的原下标00101(5)变成了10101(5 + 16) = 21。因此我们可以总结：

通过e.hash & oldCap（本例中的10000）计算得出新n- 1多出的那个高位1和key的hash值对应的比特相与是否多一个1。比如本例中：旧桶长度n为16（10000）扩容后新桶长度n'变为（100000），那么n'  -1 = 11111，而n-1 = 1111，e.hash & oldCap算出来的是多出来的那个1 & hash值对应的字节（key1是0，key2是1），因此这多出来的1对key2的新桶下标产生了影响，变成了原索引 + oldCap。

此外resize方法的两个版本的不同点还在于：

- 在JDK1.7中，新节点的在新桶链表顺序是旧桶链表里的倒序。例如原数组下标为2的节点有a->b->c，那么假设这三个节点计算出的下标不变，则放入新数组的顺序为c->b->a。
- JDK1.8则是正序的。





# 参考资料

[知乎问题：HashMap中的hash实现。 答主：二大王](https://www.zhihu.com/question/51784530)

[Java 8系列之重新认识HashMap](https://tech.meituan.com/2016/06/24/java-hashmap.html)

[深入理解HashMap（jdk8）](https://juejin.im/post/5d37b5475188251b4b32b993)