# ArrayList总结
（1）ArrayList是一个容量能动态增加的动态数组，使用默认构造方法初始化出来的容量是10。

（2）ArrayList 允许空值和重复元素，当往 ArrayList 中添加的元素数量大于其底层数组容量时，其会通过扩容机制重新生成一个更大的数组。ArrayList扩容的长度是原长度的1.5倍

（3）ArrayList 是非线程安全类。

# 成员变量和构造函数
## 继承体系和成员变量

（1）ArrayList 继承于AbstractList，实现了List，表明它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能。

（2）ArrayList 实现了标志接口RandomAccess，表明它支持快速随机访问。

（3）ArrayList 实现了Cloneable 接口，覆盖了函数 clone()，能被克隆。

（4）ArrayList 实现java.io.Serializable 接口，表明它能通过序列化传输。


```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    private static final long serialVersionUID = 8683452581122892189L;
    // 默认初始容量大小
    private static final int DEFAULT_CAPACITY = 10;

    //空数组（用于空实例）
    private static final Object[] EMPTY_ELEMENTDATA = {};

     //用于默认大小空实例的共享空数组实例。
      //我们把它从EMPTY_ELEMENTDATA数组中区分出来，
      //以知道在添加第一个元素时容量需要增加多少。
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    //保存ArrayList数据的数组
    transient Object[] elementData;  

    //ArrayList 所包含的元素个数
    private int size;
    //要分配的最大数组大小
     private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
}
```



![](https://user-gold-cdn.xitu.io/2020/5/4/171ddcee45a8f951?w=672&h=384&f=png&s=36857)


## 构造函数


```Java
    /*
    	给定容量大小initialCapacity的构造参数：
    		（1）若initialCapacity大于0，则创建initialCapacity大小的数组
    		（2）若initialCapacity等于0，则数组为空
    */
	public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            //创建指定容量initialCapacity的数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            //创建空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
        }
    }

    /**
     *	默认构造函数：数组为空
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
    
    /*
		参数为集合c的的构造函数：
			将c的数组赋值给elementData。
				（1-1）如果elementData的长度不等于0，并且如果elementData不是object类型数组，那么将其转成Object数组
				（1-2）否则elementData为空数组
    */
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        //如果指定集合元素个数不为0
        if ((size = elementData.length) != 0) {
            // （1）考虑到返回的可能不是Object类型的数组，
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size,
                                                Object[].class);
        } else {
            // 用空数组代替
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

**为什么在（1）处要判断elementData.getClass() != Object[].class**

首先先介绍一下数组的类型：

```java
    @Test
    public void test() {
        System.out.println(new String[2].getClass());
        System.out.println(new int[2].getClass());
        System.out.println(new ArrayList<>().getClass());
        System.out.println(new ArrayList<>().toArray().getClass());
        /*
            class [Ljava.lang.String;
            class [I
            class java.util.ArrayList
            class [Ljava.lang.Object;
            
         */
    }
```

然后实现Collection接口的类可能会重写toArray()方法，此时的toArray方法返回的可能就不是Object数组。



# add方法
add方法在添加元素前，需要判断是否需要扩容
```java
//将指定元素添加到list的末尾
public boolean add(E e) {
    //添加元素后可能导致容量不够，所以需要在添加之前进行判断是否扩容
    ensureCapacityInternal(size + 1);   
    elementData[size++] = e;
    return true;
}
```

## ensureCapacityInternal方法

设置最小容量minCapacity的值：如果elementData为空数组，则minCapacity的值为（默认容量10，minCapacity）里的最大值。

```java
   //若ArrayList是无参构造初始化的，则此时minCapacity = 1
    private void ensureCapacityInternal(int minCapacity) {
		//elementData是空数组的情况：
        //(1)无参构造；(2)给定容量但容量是0的有参构造
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            //无参和有参构造下的minCapacity取值10，否则是预计添加元素后的数组长度      
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
```

## ensureExplicitCapacity方法

如果最小容量minCapacity大于当前数组长度，则进行扩容方法。

```java
private void ensureExplicitCapacity(int minCapacity) {
    modCount++; 
    //如果是无参构造则肯定扩容；如果是有参构造则则判断给定的数组容量是否小于10
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);//执行扩容的方法
}

```

**modCount变量代表了什么？**
在使用迭代器遍历ArrayList里的数组时，modCount变量用于检查数组里的元素数量是否发生变化，这主要是在多线程环境中使用：防止一个线程迭代遍历的同时，另一个线程修改了这个列表的结构。如果modCount变量变化了，迭代器就抛出异常ConcurrentModificationException。
可以参考如下代码，它会抛出异常。

```java
ArrayList<Integer> list = new ArrayList<Integer>();
list.add(10);

Iterator<Integer> iterator = list.iterator();
while(iterator.hasNext()){
    Integer integer = iterator.next();
     if(integer==10)
         list.remove(integer);   
}
```

此处当我们remove掉唯一的元素后，size变为了0，而此时Iterator的游标cursor是 1 ，
在ArrayList迭代器的hasNext()方法中：

```java
    public boolean hasNext() {
         return cursor != size();
    }
```

cursor确实不等于size，因此还会进行下一次循环。如果我们不通过modCount和expectedModCount(创建迭代器的时候将当时的modCount赋值给expectedModCount)判断，则程序会报ArrayIndexOutOfBoundsException错误。JDK只抛出使用者造成的错误.



## grow方法

（1）将新容量newCapacity计算为旧容量的1.5倍，如果newCapacity 小于minCapacity（值为10或者预计添加元素后的长度），则newCapacity =minCapacity

（2）如果newCapacity 大于MAX_ARRAY_SIZE，则调用`hugeCapacity`方法取值。

（3）扩展数组，并将原数组中的元素拷贝

```java
/*
    考虑到不同的JVM会加入一下数据头，当扩容后的容量大于MAX_ARRAY_SIZE，
    我们会去比较最小需要容量和MAX_ARRAY_SIZE。
*/
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

private void grow(int minCapacity) {
    // oldCapacity为旧数组的容量
    int oldCapacity = elementData.length;
    // newCapacity为新数组的容量，它是旧容量的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 如果新容量小于最小容量，则按最小容量进行扩容
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //如果新容量大于MAX_ARRAY_SIZE，使用hugeCapacity方法比较
    //最小容量和MAX_ARRAY_SIZE
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 扩展新数组，然后将原数组中的元素拷贝
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

### hugeCapacity方法

如果最小容量大于MAX_ARRAY_SIZE，则返回Integer.MAX_VALUE ，否则返回MAX_ARRAY_SIZE。

```java
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) 
        throw new OutOfMemoryError();
    /*
    MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    对minCapacity和MAX_ARRAY_SIZE进行比较：
        若minCapacity大，将Integer.MAX_VALUE作为新数组大小
        若MAX_ARRAY_SIZE大，将MAX_ARRAY_SIZE作为新数组大小
    */
    return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE 
                                    : MAX_ARRAY_SIZE;
}
```

以无参初始化的ArrayList为例，它的容量为10：当添加第11个元素时，就会进入grow方法：计算出newCapacity = 15，后两个判断条件都不满足，因此数组扩容为15，size变为11。



## ArrayList的线程不安全

ArrayList的线程不安全性主要体现在：

添加元素操作不是原子性的：`elementData[size++] = e;`，这段代码分为两步：

- elementData[size] = e;
- size++;

我们来假设在一个多线程环境下，当前列表长度为0：

- 线程 A 执行完 `elementData[size] = e;`之后挂起。A 把 "a" 放在了下标为 0 的位置。此时 size = 0。
- 线程 B 执行 `elementData[size] = e;` 因为此时 size = 0，所以 B 把 "b" 放在了下标为 0 的位置，于是刚好把 A 的数据给覆盖掉了。
- 线程 B 将 size 的值增加为 1，线程 A 将 size 的值增加为 1。

这样在线程 A 和线程 B 都执行完之后理想情况下应该是 "a" 在下标为 0 的位置，"b" 在标为 1 的位置。而实际情况确是下标为 0 的位置为 "b"，下标为 1 的位置啥也没有。



# add(int index,E element)方法

（1）首先判断indx是否在[0, size - 1]区间中

（2）检查是否需要扩容，然后[index, size-1]的元素都向后移一位

（3）将新元素element插入至 index 处，然后size++。


```java
//在元素序列 index 位置处插入元素element
public void add(int index, E element) {
    rangeCheckForAdd(index); //判断index是否在[0, size - 1]区间中
    // 检测是否需要扩容
    ensureCapacityInternal(size + 1);  
    // [index, size-1]的元素都向后移一位
    System.arraycopy(elementData, index, elementData, index + 1, size - index);
    // 将新元素插入至 index 处
    elementData[index] = element;
    size++;
}
//Java可以通过JNI来调用其他语言（主要还是C/C++语言）编写的方法(本地方法)
//src：原数组；srcPos：源数组要赋值的起始位置；dest：目标数组；length：复制的长度
public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```

# remove方法
ArrayList支持两种删除元素的方式：

**（1）remove(int index)： 按照下标删除**


```java
public E remove(int index) {
    rangeCheck(index); //校验下标是否合法
    modCount++;//修改list结构，就需要更新这个值
    E oldValue = elementData(index); //根据下标查找值
	
    int numMoved = size - index - 1;//index后面有多少个元素
    if (numMoved > 0)
        //[index +1, size-1]的元素左移一位
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    //移动后，原数组中size-1位置为null
    elementData[--size] = null; // clear to let GC do its work
    //返回旧值
    return oldValue;
}

```

**（2）remove(Object o)：按照元素删除，会删除和参数匹配的第一个元素**


```java
public boolean remove(Object o) {
    //如果元素是null 遍历数组移除第一个null
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        //找到元素对应的下标 调用下标移除元素的方法
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}
//根据下标来移除元素
private void fastRemove(int index) {
  modCount++;
  int numMoved = size - index - 1;//计算index后面有多少个元素
  if (numMoved > 0)
    System.arraycopy(elementData, index+1, elementData, index,
                     numMoved);
  elementData[--size] = null; // clear to let GC do its work
}

```



# 参考资料

[java ArrayList源码中注释的疑问](https://segmentfault.com/q/1010000000327000)

[知乎问题：arraylist等记录修改次数modCount有什么作用？答主：wuxinliulei](https://www.zhihu.com/question/24086463)

[ArrayList源码分析（扩容机制jdk8）](https://juejin.im/post/5d42ab5e5188255d691bc8d6#heading-0)

[JavaGuide](https://github.com/Snailclimb/JavaGuide)