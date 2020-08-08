# CAS

## **什么是CAS**

CAS是一种原子操作，一种系统源语，一条CPU的原子指令。CAS的全称是比较并交换（Compare And Swap），它有三个值：

- V：要更新的变量(var)
- E：预期值(expected)
- N：新值(new)

该操作的流程是：判断V是否等于E，若是，则将V的值设置为N；如果不是，说明已经有其它线程更新了V，则当前线程放弃更新，什么都不做。

> 你可能会认为在准备将V更新为N时，其他线程会修改V值。但这不会发生。因为CAS是一种原子操作，它是一种系统原语，是一条CPU的原子指令，从CPU层面保证它的原子性
>
> **当多个线程同时使用CAS操作一个变量时，只有一个会胜出，并成功更新，其余均会失败，但失败的线程并不会被挂起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。**

## Unsafe类

Java有一个在sum.misic包中的Unsafe类，主要提供一些用于执行低级别、不安全操作的方法，如直接访问系统内存资源、自主管理内存资源等。

它有一些关于CAS的native方法：

```java
boolean compareAndSwapObject(Object o, long offset,Object expected, Object x);

boolean compareAndSwapInt(Object o, long offset,int expected,int x);

boolean compareAndSwapLong(Object o, long offset,long expected,long x);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

此外Unsafe类里面还有其它方法，比如支持线程挂起和恢复的park和unpark， LockSupport类底层就是调用了这两个方法。还有支持反射操作的allocateInstance()方法。



## 原子操作类

JDK提供了一些用于原子操作的类，在java.util.concurrent.atomic包里。

![img](https://img-blog.csdnimg.cn/20200428144349175.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)
这些类主要是 原子更新基本类型；原子更新数组；原子更新引用和原子更新字段（属性）。



### 原子更新基本类型

Atomic包提供了以下3个类来原子的更新基本类型：

- AtomicBoolean：原子更新布尔类型。
- AtomicInteger：原子更新整型。
- AtomicLong：原子更新长整型。

AtomicInteger的示例代码如下

```java
    public static void main(String[] args) throws InterruptedException {
        AtomicInteger ai = new AtomicInteger(1);
        System.out.println(ai.getAndIncrement()); //原子性的将当前值加1，返回旧值1.
        System.out.println(ai.get());      		 //输出2
    }
```

**getAndIncrement方法**

此处我们来介绍`getAndIncrement`方法

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private volatile int value;
    //记录value变量在AtomicInteger对象上内存偏移量。通过valueOffset直接在内存中修改value的值
    private static final long valueOffset;
    
    static {
        try {
            //通过unsafe.objectFieldOffset方法从AtomicInteger对象中获取value的偏移量
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
    
    public final int getAndIncrement() {
         return unsafe.getAndAddInt(this, valueOffset, 1);
    }
}
```

再进入`getAndAddInt`方法。此处方法参数var1是调用`getAndIncrement`方法的AtomicInteger对象，var2是其成员变量：偏移量valueOffset

```java
public final class Unsafe {
	public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
		//如果var1里的成员变量value和var5相等，那么将它更新为var5 + var4。否则采用自旋方式继续进行CAS操作。
        //这两个操作看似是两个步骤，但在JNI里是借助于一个CPU指令完成的，因此还是原子操作。
        return var5;	//返回旧值
    }
}
```



**如何原子的更新其他基本类型**

可以发现，Atomic只提供了3种基本类型的原子更新，而Unsafe类也只提供了如下3种CAS方法：

```java
public final native boolean compareAndSwapObject(Object o,long offset,Object expected, Object x);
 
public final native boolean compareAndSwapInt(Object o, long offset,
int expected, int x);
 
public final native boolean compareAndSwapLong(Object o, long offset,
long expected, long x);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

我们可以参考下AtomicBoolean类的转换，如下所示，它先把把Boolean转换成整 型，再使用compareAndSwapInt进行CAS。因此对于char，float和double等变量，我们也可以使用类似的思路来实现。

```java
    public final boolean compareAndSet(boolean expect, boolean update) {
        int e = expect ? 1 : 0;
        int u = update ? 1 : 0;
        return unsafe.compareAndSwapInt(this, valueOffset, e, u);
    }
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### 原子更新数组类型

通过原子的方式更新数组里的某个元素，Atomic包提供了以下3个类

- AtomicIntegerArray：原子更新整型数组里的元素。
- ·AtomicLongArray：原子更新长整型数组里的元素。
- ·AtomicReferenceArray：原子更新引用类型数组里的元素。

下面是AtomicIntegerArray的示例代码，可以发现，数组value通过构造方法传递进去，然后AtomicIntegerArray会将当前数组复制一份，所以当AtomicIntegerArray对内部的数组元素进行修改时，不会影响传入的数组。

```java
    public static void main(String[] args) throws InterruptedException {
        int[] array = new int[]{1, 2};
        AtomicIntegerArray ai = new AtomicIntegerArray(array);
        ai.getAndSet(0, 3);
        System.out.println(ai.get(0));  //输出3
        System.out.println(array[0]);   //输出1
    }
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### 原子更新引用类型

原子更新基本类型的AtomicInteger 只能更新一个变量，如果要原子更新多个变量，就需 要使用这个原子更新引用类型提供的类。Atomic包提供了以下3个类

- **AtomicReference**：原子更新引用类型。
- **AtomicReferenceFieldUpdater**：原子更新引用类型里的字段。
- **AtomicMarkableReference**：原子更新带有标记位的引用类型。可以原子更新一个布尔类 型的标记位和引用类型。构造方法是AtomicMarkableReference（V initialRef，boolean initialMark）。

下面使用一下AtomicReference：

```java
public class Test {
    public static class User {
        private String name;
        private int agel
 
        public User(String name, int agel) {
            this.name = name;
            this.agel = agel;
        }
 
        //下面省略了各个字段的get/set方法。
    }
 
    public static void main(String[] args) throws Exception{
            AtomicReference<User> userRef = new AtomicReference<>();
            User user = new User("conman", 15);
            userRef.set(user);
            User updateUser = new User("new conman", 17);
            userRef.compareAndSet(user, updateUser);
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### 原子更新字段类

如果希望原子地更新某个类的某个字段，可以使用原子更新字段类，Atomic包提供 了以下3个类进行原子字段更新。 

- ·AtomicIntegerFieldUpdater：原子更新整型的字段的更新器。

- ·AtomicLongFieldUpdater：原子更新长整型字段的更新器。
- ·AtomicIntegerFieldUpdater：原子更新整型的字段的更新器。
- ·AtomicStampedReference：原子更新带有版本号的引用类型。该类将整数值与引用关联起 来，可用于原子的更新数据和数据的版本号，可以解决使用CAS进行原子更新时可能出现的 ABA问题。

下面我们使用一下AstomicIntegerFieldUpdater类，需要说明的是：更新类的字段必须使用public volatile修饰符；每次使用都必须用静态方法newUpdater()方法来创建一个更新器，并设置想要更新的类和属性。

```java
public class Test {
    public static class User {
        private String name;
        public volatile int age;        // public volatile
 
        public User(String name, int agel) {
            this.name = name;
            this.age = agel;
        }
 
       //下面省略了各个字段的get/set方法。
    }
 
    public static void main(String[] args) throws Exception{
        // 创建原子更新器，并设置需要更新的对象类和对象的属性
        AtomicIntegerFieldUpdater<User> userFie = AtomicIntegerFieldUpdater.newUpdater(User.class,
                "age");
        User conan = new User("conan", 10);
        //          增加1岁：输出的是旧值 10
        System.out.println(userFie.getAndIncrement(conan));
        //        输出新值 11
        System.out.println(userFie.get(conan));
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



## CAS实现原子操作的三大问题

### ABA问题

一个值原来是A，变成了B后，又变回了A，此时该值更新了两次，但CAS是察觉不到变化的。

因此我们可以在变量前面追加**版本号或者时间戳**。从JDK 1.5开始，JDK的atomic包里提供了一个类`AtomicStampedReference`类来解决ABA问题，该类的weakCompareAndSet方法可以检查当前引用是否等于预期引用，并且检查当前标志是否等于预期标志，如果二者都相等，才使用CAS设置为新的值和标志。

```java
    public boolean weakCompareAndSet(V   expectedReference,
                                     V   newReference,
                                     int expectedStamp,
                                     int newStamp) {
        return compareAndSet(expectedReference, newReference,
                             expectedStamp, newStamp);
    }
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### 循环时间长开销大

CAS大多与自旋结合使用，如果自旋CAS长时间不成功，会占用大量的CPU资源。

该问题的解决思路是让JVM支持处理器提供的**pause指令**，pause指令能让自旋失败时cpu睡眠一小段时间再继续自旋，从而使得读操作的频率低很多,为解决内存顺序冲突而导致的CPU流水线重排的代价也会小很多。



### 只能保证一个共享变量的原子操作

该问题有两种解决方案：

（1）使用AtomicReference类，该类可以保证对象之间的原子性。因此我们可以把多个变量放到一个对象里面进行CAS操作；

（2）使用锁。锁内的临界区代码可以保证只有当前线程能操作。



# 参考资料

[深入浅出Java多线程](http://concurrent.redspider.group/RedSpider.html)