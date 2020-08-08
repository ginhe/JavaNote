# SimpleDateFormat的线程不安全性

现在有一个需求：在多线程环境下去格式化时间。那么我们就需要SimpleDateFormat 类

```java
public class Thread2 {
    public static ExecutorService pool = Executors.newFixedThreadPool(10);
    //类变量
    static SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
	//每个线程获取时间的方法
    private String handlerDate(int seconds) {
        Date date = new Date(1000 * seconds);
        return sdf.format(date);
    }

    @Test
    public void testThreadLocal() throws InterruptedException{
        for(int i = 0; i < 1000; i++) {
            final int curIndx = i;
            pool.submit(new Runnable() {
                @Override
                public void run() {
                    String date = new Thread2().handlerDate(curIndx);
                    System.out.println(date);
                }
            });
        }
        Thread.sleep(5000);
        pool.shutdown();
    }
}
```

我们会发现输出里出现了重复的时间格式化内容，这是因为SimpleDateFormat是一个线程不安全的类，其实例对象在多线程环境下作为共享数据，会发生线程不安全问题。

<img src="https://user-gold-cdn.xitu.io/2020/7/20/1736b4f32d5bafbb?w=919&amp;h=757&amp;f=png&amp;s=33476" style="zoom:67%;" />

我们可以通过synchronized关键字来限制handlerDate方法：

```java
private String handlerDate(int seconds) {
    
    Date date = new Date(1000 * seconds);
    String format;
    synchronized (ThreadLocalUsage02.class) {
        format = sdf.format(date);
    }
    return format;
}
```

不过锁会带来性能的下降，因此我们可以使用其他的方法来解决：ThreadLocal。



# ThreadLocal

ThreadLocal不是线程，更不是本地线程，而是Thread的局部变量，它是每个线程独享的本地变量，每个线程都有自己的ThreadLocal，它们是线程隔离的。

## 示例1

下面来用ThreadLocal来解决SimpleDateFormat类的问题

```java
public class ThreadSafeFormatter {

    public static ThreadLocal<SimpleDateFormat> dateFormatThreadLocal = new ThreadLocal<SimpleDateFormat>() {
        @Override
        protected SimpleDateFormat  initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd HH hh:mm:ss");
        }
    };
}
```

修改上例`handlerDate`方法

```java
    private String handlerDate(int seconds) {
        Date date = new Date(1000 * seconds);
        //获取sdf
        SimpleDateFormat sdf = ThreadSafeFormatter.dateFormatThreadLocal.get();
        return sdf.format(date);
    }
```

测试方法不变，运行后会发现不会出现重复的问题。

ThreadLocal将SimpleDateFormat对象用ThreadLocal包装了一层，使得多个线程内部都有一个SimpleDateFormat对象副本，每个线程使用自己的SimpleDateFormat，这样就不会产生线程安全问题了。

## 示例2

假设我们有一个学生类

```java
public class Student {
    String name;
    public Student(String name) {
        this.name = name;
    }
}
```

我们需要将某一学生信息在线程内的所有方法中共享，因此我们可以将Student对象作为方法参数来进行传递。

```java
class Service1  {

    public void process(Student student) {
        System.out.println(student.name);
    }

}

class Service1  {

    public void process(Student student) {
        System.out.println(student.name);
    }

}

class Service1  {

    public void process(Student student) {
        System.out.println(student.name);
    }

}
```

但这样做会产生代码冗余问题，并且可维护性差。此外你可以使用HashMap来存储学生信息，这样后续的使用直接get方法即可。

<img src="https://user-gold-cdn.xitu.io/2020/7/20/1736b8200899ecfb?w=841&amp;h=242&amp;f=png&amp;s=57194" style="zoom:67%;" />

但是在**多线程环境下**需要使用ConcurrentHashMap，但它所使用的CAS和锁机制会产生**性能**问题。

因此我们可以使用ThreadLocal来实现不同方法的资源共享

```java
	 @Test
    public void testThreadLocal2() {
        new Service1().process();
    }

    static class StudentThreadLocal {
        public static ThreadLocal<Student> studentLocal = new ThreadLocal<>();
    }

    class Service1 {
        //设置名字并往后传递
        public void process() {
            Student stu = new Student("gua");
            //将User对象存储到 holder 中
            StudentThreadLocal.studentLocal.set(stu);
            new Service2().process();
        }
    }

    class Service2 {

        public void process() {
            Student stu = StudentThreadLocal.studentLocal.get();
            System.out.println("Service2拿到学生名: " + stu.name);
            new Service3().process();
        }
    }

    class Service3 {

        public void process() {
            Student stu = StudentThreadLocal.studentLocal.get();
            System.out.println("Service3拿到学生名: " + stu.name);
        }
    }

```



## 原理

首先Thread类里维护了ThreadLocalMap成员变量

```java
public class Thread implements Runnable {
    ThreadLocal.ThreadLocalMap threadLocals = null;

    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
}
```

ThreadLocal类里有一个静态内部类ThreadLocalMap。而在ThreadLocalMap类里有一个Entry类，它继承了ThreadLocal类的弱引用，并将其作为key，value为Object类型。

```java
public class ThreadLocal<T> {	
	static class ThreadLocalMap {
        
        static class Entry extends WeakReference<ThreadLocal<?>> {
            Object value;
            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
        
        // 默认的数组初始化容量
        private static final int INITIAL_CAPACITY = 16;
        // Entry数组，大小必须为2的幂
        private Entry[] table;
        // 数组内部元素个数
        private int size = 0;
        // 数组扩容阈值，默认为0，创建了ThreadLocalMap对象后会被重新设置
        private int threshold;
        
        //构造方法
        ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            // 初始化Entry数组，大小 16
            table = new Entry[INITIAL_CAPACITY];
             // 用第一个键的哈希值对初始大小减一取模得到索引，和HashMap计算桶下标的原理一样。
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            // 将Entry对象存入数组指定位置
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            // 初始化扩容阈值，第一次设置为10
            setThreshold(INITIAL_CAPACITY);
        }
    }
}
```

**总结来说：**

- 线程类**Thread**内部有成员变量：类型是**ThreadLocalMap**的threadLocals
  - **ThreadLocalMap**类里有一个内部类Entry，ThreadLocalMap持有属性：Entry数组table负责存储键值对，其中key是ThreadLocal的弱引用，value是要存储的对象。
  - **ThreadLocalMap**是ThreadLocal的内部类，我们在前面通过ThreadLocal对象来使用的set，get方法主要调用了其内部类**ThreadLocalMap**的方法。



### **set方法**

该方法获取当前线程的成员变量threadLocals，并将threadLocals - value作为键值对存储在Entry数组talbe里

```java
public class ThreadLocal<T> {	
	public void set(T value) {
        //获取调用此方法的线程
        Thread t = Thread.currentThread();
        //获取t的成员变量threadLocals(ThreadLocal.ThreadLocalMap threadLocals)
        ThreadLocalMap map = getMap(t);
        //若map不为空，则说明当前线程内部已经有ThreadLocalMap对象，则将成员变量threadLocals对象作为键，存入的value作为值存储到ThreadLocalMap中
        if (map != null)		
            map.set(this, value);
        // 否则创建一个ThreadLocalMap对象并将值存入到该对象中，并赋值给当前线程的threadLocals成员变量
        else
            createMap(t, value);
    }

   ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

    // 创建一个ThreadLocalMap对象并将值存入其中，并赋值给当前线程的threadLocals成员变量
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
}
```

下面是存储键值对的set方法。它将键值对存储在Entry数组table里。

```java
private void set(ThreadLocal<?> key, Object value) {

    Entry[] tab = table;
    int len = tab.length;
    //计算当前ThreadLocal对象作为键在Entry数组中的下标索引
    int i = key.threadLocalHashCode & (len-1);
	//获取到指定下标的Entry对象，如果不为空，则进入到for循环体内，
    for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
		//如果是同一对象，则设置新值，返回
        if (k == key) {
            e.value = value;
            return;
        }
		//如果不是同一对象，则判断当前Entry的key是否失效，如果失效，则直接将失效的key和值进行替换。
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }
	//通过循环中的nextIndex找到下一个合适的位置并放入键值对
    tab[i] = new Entry(key, value);
    int sz = ++size;
    //判断是否要扩容
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}

//如果当前i是最后一个，则从数组头部重新找。这种方式是开放寻址法的应用（而HashMap采用的是链表方式存储哈希冲突）
private static int nextIndex(int i, int len) {
      return ((i + 1 < len) ? i + 1 : 0);
}
```



### get方法

我们通过该方法获取存储在当前线程的成员变量threadLocals里的值

```java
    public T get() {
        Thread t = Thread.currentThread();
        //当前线程t的成员变量threadLocals
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            //获取En
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        //如果没找到或Entry数组为空，则进行初始化：将当前ThreadLocal对象作为key，null作为value。
        //然后存入到当前线程的ThreadLocalMap对象中
        return setInitialValue();
    }

    private T setInitialValue() {
        T value = initialValue();	//返回null
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);		//返回当前线程的成员变量threadLocals
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }
```



### remove方法

```java
// 获取当前线程的成员变量threadLocals，它将作为key来查找Entry数组里的value值。
     public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }

     private void remove(ThreadLocal<?> key) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);
            for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
                if (e.get() == key) {
                    e.clear();
                    expungeStaleEntry(i);
                    return;
                }
            }
        }
```



## ThreadLocalMap内存泄露问题

首先简单介绍一下java的四大引用

（1）**强引用**：java默认的引用类型。比如`String str = new String("jnju");`，其中str就是一个强引用。一个对象如果具有强引用那么只要这种引用还存在就不会被回收。

（2）**软引用**：如果一个对象具有软引用，在JVM发生内存溢出之前（即内存充足够使用），是不会GC这个对象的；只有到JVM内存不足的时候才会调用垃圾回收期回收掉这个对象。软引用和一个引用队列联合使用，如果软引用所引用的对象被回收之后，该引用就会加入到与之关联的引用队列中。

（3）**弱引用**：如果一个对象只具有弱引用，那么这个对象就会被垃圾回收器回收掉（被弱引用所引用的对象只能生存到下一次GC之前，当发生GC时候，无论当前内存是否足够，弱引用所引用的对象都会被回收掉）。

弱引用也是和一个引用队列联合使用，如果弱引用的对象被垃圾回收期回收掉，JVM会将这个引用加入到与之关联的引用队列中。若引用的对象可以通过弱引用的get方法得到，当引用的对象被回收掉之后，再调用get方法就会返回null。

（4）**虚引用**：所有引用中最弱的一种引用，其存在就是为了将关联虚引用的对象在被GC掉之后收到一个通知。

我们再回到存放键值对的类Entry：

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
       Object value;
       Entry(ThreadLocal<?> k, Object v) {
             super(k);
             value = v;
        }
}
```

由此可知：ThreadLocal的弱引用k通过构造方法传递给了Entry类的父类WeakReference的构造方法，可以理解ThreadLocalMap中的键是ThreadLocal的弱引用。

> **内存泄漏是指某个对象不会再被使用，但是该对象的内存却无法被收回**

正常情况下 当Thread运行结束后，ThreadLocal中的value会被回收，因为没有任何强引用了。

但是在非正常情况下，当Thread一直在运行始终不结束，强引用就不会被回收，此时存在以下调用链 Thread-->ThreadLocalMap-->Entry(key为null)-->value 因为调用链中的 value 和 Thread 存在强引用，所以**value无法被回收**，就有可能出现**OOM**。

JDK的设计考虑到了这个问题，所以在set()、remove()、resize()方法中会扫描到key为null的Entry，并且把对应的value设置为null，这样value对象就可以被回收。但是只有在调用set()、remove()、resize()这些方法时才会进行这些操作，如果没有调用这些方法并且线程不停止，那么调用链就会一直存在，所以可能会发生内存泄漏。

因此**z在使用完ThreadLocal后，要调用remove()方法。**



# 参考资料

[ThreadLocal](https://mp.weixin.qq.com/s/bcH2pL06J5udBWedE1tbCA)