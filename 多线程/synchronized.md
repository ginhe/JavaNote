# synchronized

## **非线程安全**

非线程安全发生 在多个线程对同一个对象中的实例变量进行并发访问，取到的数据是被修改过的，也就是“脏读”。而线程安全是指获得的实例变量的值是经过同步处理的。

在下面的示例代码中，自定义类HasSelfNum对象处理的变量num是方法的本地变量，此时是线程安全的。

```java
    static class HasSelfNum {
        public void addNum(String str) {
            try {
                int num = 0;
                if(str.equals("a")) {
                    num = 100;
                    System.out.println("a 已放置");
                    Thread.sleep(2000);
                }else {
                    num = 200;
                    System.out.println("b 已放置");
                }
                System.out.println(str+ "num：" + num);
            }catch (InterruptedException e) {e.printStackTrace();}
        }
    }

    static class ThreadA extends Thread{
        private HasSelfNum numRef;

        public ThreadA(HasSelfNum numRef) {
            this.numRef = numRef;
        }

        @Override
        public void run() {
            numRef.addNum("a");
        }
    }


    static class ThreadB extends Thread{
        private HasSelfNum numRef;

        public ThreadB(HasSelfNum numRef) {
            this.numRef = numRef;
        }

        @Override
        public void run() {
            numRef.addNum("b");
        }
    }

    public static void main(String[] args) {
            HasSelfNum numRef = new HasSelfNum();
            ThreadA ta = new ThreadA(numRef);
            ThreadB tb = new ThreadB(numRef);

            ta.start();
            tb.start();
    }
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

而如果num变量不是方法里的本地变量，而是HasSelfNum对象的实例变量的话，则有可能出现“非线程安全”。

```java
    static class HasSelfNum {
        private  int num = 0;       //变量num不是方法的本地变量
        public void addNum(String str) {
            try {
                if(str.equals("a")) {
                    num = 100;
                    System.out.println("a 已放置");
                    Thread.sleep(2000);
                }else {
                    num = 200;
                    System.out.println("b 已放置");
                }
                System.out.println(str+ "num：" + num);
            }catch (InterruptedException e) {e.printStackTrace();}
        }
    }

    static class ThreadA extends Thread{
        private HasSelfNum numRef;

        public ThreadA(HasSelfNum numRef) {
            this.numRef = numRef;
        }

        @Override
        public void run() {
            numRef.addNum("a");
        }
    }


    static class ThreadB extends Thread{
        private HasSelfNum numRef;

        public ThreadB(HasSelfNum numRef) {
            this.numRef = numRef;
        }

        @Override
        public void run() {
            numRef.addNum("b");
        }
    }

    public static void main(String[] args) {
            HasSelfNum numRef = new HasSelfNum();
            ThreadA ta = new ThreadA(numRef);
            ThreadB tb = new ThreadB(numRef);

            ta.start();
            tb.start();
    }
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

输出：

![img](https://img-blog.csdnimg.cn/20200427200410475.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

由此可见，若多个线程同时通过一个没有同步的方法来操作某一对象中的实例变量，则可能出现“非线程安全”。



## 使用Synchronized关键字

我们可以通过Synchronized关键字来给一段代码或一个方法上锁，而Java中的每一个对象都可以作为一个锁。

在上面代码的基础上，我们为HasSelfNum类的方法加上锁，即可实现同步方法。

```java
public class HasSelfNum {
    int num = 0;
    synchronized public void addNum(String str) {
        try {
            if(str.equals("a")) {
                num = 100;
                System.out.println("a 已放置");
                Thread.sleep(2000);
            }else {
                num = 200;
                System.out.println("b 已放置");
            }
            System.out.println(str+ "num：" + num);
        }catch (InterruptedException e) {e.printStackTrace();}
    }
```

当一个线程试图访问锁住的代码时，它必须得先得到锁，而在它退出或抛出异常时必须释放锁。



### synchronized锁方法

当使用synchronized锁方法时，是将调用该方法的对象作为一把锁，比如说上例代码中的HasSelfNum对象。

若一个对象在两个线程中分别调用synchronized方法和非同步方法，这两个方法是不会产生互斥的。

**synchronized锁重入**

当一个线程得到一个对象锁后，再次请求此对象锁是可以再次得到的。也就是说，在一个同步方法块的内部调用本类的其他synchronized方法，是永远可以得到的。

```java
public class HasSelfNum {
    synchronized public void service1() {
        System.out.println(Thread.currentThread().getName() + " service1");
        service2();
    }
 
    synchronized public void service2() {
        System.out.println(Thread.currentThread().getName() + " service2");
        service3();
    }
 
    synchronized public void service3() {
        System.out.println(Thread.currentThread().getName() + " service3");
 
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

若存在父子类继承关系，则子类可以通过可重入锁调用父类的加锁方法。

**同步不能继承**

父类中被synchronized修饰的方法，子类在重写该方法时，若不加synchronized修饰，重写的方法是不会继承其父类的同步性的。



### synchronized锁类对象

当使用synchronized锁方法，则如果拿到锁的线程执行的是一个长时间的任务，则另一个线程需要等待较长时间。因此我们可以synchronized锁住某一代码块而不是整个方法，这样可以让等待锁的另一个线程先执行 没有被锁定的代码段，提高效率。

我们可以将任意类对象作为锁，如下所示。

```java
Object lock = new Object();
public class Service {
    private String data1;
    private String data2;
     public void service1() {
        try {
            System.out.println("开始任务-------------------");
            Thread.sleep(3000);
            String tmp1 = "处理任务后返回的值1：" + Thread.currentThread().getName();
            String tmp2  = "处理任务后返回的值2：" + Thread.currentThread().getName();

            synchronized(lock ) {            //锁代码段
                data1 = tmp1;
                data2 = tmp2;
            }
            System.out.println(data1);
            System.out.println(data2);
            System.out.println("结束任务-------------------");
        }catch (InterruptedException e) { e.printStackTrace(); }
    }
```



### synchronized(String)的特性

String类的初始化两种，一种是new String()，还有一种是直接赋值。如下代码所示，由于JVM中字符串常量池的原因，这两种初始化方式判断是否相等的结果是true。

```java
    public static void main(String[] args) throws InterruptedException {
        String a = "a";
        String b = "a";
        System.out.println(a == b); // 输出true
    }
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

因此如果我们将String作为对象锁，需要注意其初始化，如下代码所示，传入的字符串参数都是通过直接赋值来初始化的，这样进入同步的代码段所要求的锁是相同的。

```java
public class Service {
     public  static void service1(String str) {               // 静态方法
        try {
            synchronized(str) {
                while(true) {
                    System.out.println(Thread.currentThread().getName());
                    Thread.sleep(2000);
                }
            }
 
        } catch (InterruptedException e) { e.printStackTrace(); }
    }
}
 
public class ThreadA extends Thread{
    private Service service;
 
    public ThreadA(Service service) {
        this.service = service;
    }
 
    @Override
    public void run() {
        service.service1("AA");
    }
}
 
public class ThreadB  extends Thread{
    private Service service;
 
    public ThreadB(Service service) {
        this.service = service;
    }
 
    @Override
    public void run() {
        service.service1("AA");
    }
}
 
public class Thread_Test {
    public static void main(String[] args) throws InterruptedException{
        Service service = new Service();
        ThreadA ta =  new ThreadA(service);
        ThreadB tb = new ThreadB(service);
 
       ta.setName("ta");           tb.setName("tb");
 
        ta.start();
        tb.start();
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### synchronized(this)

与synchronized方法一样，synchronized(this)代码块也是调用该方法的对象作为锁。同样，若某一对象在两个线程分别调用synchronized(this)修饰代码块的方法 和 非同步的方法，这两个方法是不会产生互斥的。

```java
public class Test {
    static class HasSelfNum {
        private String name;
        private int num;
        public void addNum(String str) {        // synchronized(this)
            try {
                System.out.println("开始任务-------------------");
                Thread.sleep(3000);
                synchronized(this) {
                    name = new String("方法a");
                    num = 111;
                }
                System.out.println(name);
                System.out.println(num);
                System.out.println("结束任务-------------------");
            }catch (InterruptedException e) { e.printStackTrace(); }
        }
    
       public void fun() {            //没有锁，执行该方法的线程无需拿到对象锁便可执行。
           System.out.println("呱");
        }
    }

}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

不过，**synchronized同步方法和synchronized(this)同步代码块**都会 阻塞调用其他synchronized同步的方法或synchronized(this)同步的代码块的线程。

**锁住的位置**

在使用锁时，我们需要注意锁住的位置是否可以实现所想要的同步。在下面的代码中，两个线程想要对共享数组中放入数据，不过数组只能放入一个数据。我们用**synchronized**锁住了CommonUtils对象的两个方法。则两个线程的执行步骤可能是：

（1）当一个线程a或b进入service1方法的判断分支，拿到CommonUtils对象锁判断数组的长度是否为零，然后释放锁后，执行后续步骤的睡眠。此时另外一个线程也进入了这个代码段，此时数组还未放入数据，因此另外一个线程也跟着进入了后续的步骤。这样就出现了两个线程都向数组存数据的现象。

```java
public class Test {
    static class CommonUtils {                //共享数据——数组
        private List list = new ArrayList();

        synchronized public void add(String data)       { list.add(data);}
        synchronized public int getSize()       { return list.size(); }
    }

    static class Service {                    //任务类
        public  void service1(CommonUtils list, String data1) {
            try {
            // 若共享的数组里没有数据，才会往数组里添加数据
                if(list.getSize() < 1) {
                    Thread.sleep(2000);            //放数据花费2s
                    System.out.println(Thread.currentThread().getName() + "准备放入数据");
                    list.add(data1);
                    System.out.println(Thread.currentThread().getName() + "结束放入数据");
                }
            }catch (InterruptedException e)        { e.printStackTrace(); }
        }
    }

    static class ThreadA extends Thread{
        private Service service;
        private  CommonUtils list;

        public ThreadA(CommonUtils list, Service service) {
            this.list = list;
            this.service =  service;
        }

        @Override
        public void run() {
            service.service1(list, "A");
        }
    }


    static class ThreadB extends Thread{
        private Service service;
        private  CommonUtils list;

        public ThreadB( CommonUtils list, Service service) {
            this.list = list;
            this.service =  service;
        }

        @Override
        public void run() {
            service.service1(list, "B");
        }
    }

        public static void main(String[] args) throws InterruptedException{
            Service service = new Service();
            CommonUtils list = new CommonUtils();
            ThreadA ta =  new ThreadA(list, service);
            ThreadB tb = new ThreadB(list, service);
            ta.setName("ta");           tb.setName("tb");
            ta.start();
            tb.start();
            Thread.sleep(6000);
            System.out.println("listSize：" + list.getSize());
    }

}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

因此我们更换锁住的代码段：

```java
public  void service1(CommonUtils list, String data1) {
      try {
         synchronized(list) {            // 此处加锁
             if(list.getSize() < 1) {
                 Thread.sleep(2000);            //取数据花费2s
                 System.out.println(Thread.currentThread().getName() + "准备放入数据");
                 list.add(data1);
                 System.out.println(Thread.currentThread().getName() + "结束放入数据");
              }
         }
      }catch (InterruptedException e)        { e.printStackTrace(); }
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### synchronized锁静态同步方法和synchronized(class)代码块

**synchronized锁静态同步方法**

关键字synchronized可以用在静态方法上，它会对当前*.java文件对应的Class类持有锁。

（1）当一个对象在两个线程中调用两个静态同步方法，两者会产生互斥。

（2）当一个对象在两个线程中分别调用一个静态同步方法和一个非静态同步方法，两者是不会产生互斥的。这是因为两个方法的锁类型不同，调用静态方法实际是类对象在调用，这两个产生的不是同一个对象锁。

**synchronized(class)或者synchronized (this.getClass())**

我们可以通过**synchronized(class)**来锁代码段，其遇到的情况处理与**synchronized锁静态同步方法**一样。



## 锁的内存语义

**锁的获取**

当线程获取锁时，JMM会把该线程对应的本地内存置为无效，从而使被锁保护的临界区代码必须从主内存中读取共享变量。

<img src="https://img-blog.csdnimg.cn/20200428190908213.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70" alt="img" style="zoom:67%;" />![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**锁的释放**

当线程释放锁时，JMM会把线程的本地内存中的共享变量刷新到主内存中。

<img src="https://img-blog.csdnimg.cn/20200428190955642.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70" alt="img" style="zoom:67%;" />![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

对于锁的获取和释放，我们可以总结为：

（1）线程A释放一个锁，实质是线程A向接下来将要获取这个锁的某个线程发出了 线程A对共享变量做了修改的消息；

（2）线程B获取一个锁，实质是线程B接收了之前某个线程发出的（在释放这个锁之前对共享变量所做修改的）消息。

（3）线程A释放锁，随后线程B获取这个锁，这个过程实质上是线程A通过主内存向线程B发送消息。



## synchronized原理

（1） **synchronized 同步语句块**

```java
public void fun() {
     synchronized (this) {
         System.out.println("同步方法");
     }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

我们可以通过JDK自带的 javap 命令查看 SynchronizedDemo 类的相关字节码信息：

```
javac F:\Thread_learn\src\Tools\Test.java
//对应类目录
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

然后执行

```
javap -c -s -v -l  F:\Thread_learn\src\Tools\Test.class
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

显示结果如下。**synchronized 同步语句块的实现使用的是 monitorenter 和 monitorexit 指令，其中 monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置。** 

线程试图获取锁就是试图获取 存在于每个Java对象的对象头的monitor：

- 如果monitor的进入数为0，则线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者；

- 如果线程已经占有该monitor，只是重新进入，那么进入monitor的进入数加1；

- 如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权；

  

![img](https://img-blog.csdnimg.cn/20200501193816232.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

因此我们可以看出**Synchronized的语义底层是通过一个monitor的对象来完成，其实wait/notify等方法也依赖于monitor对象，这就是为什么只有在同步的块或者方法中才能调用wait/notify等方法**



（2）**synchronized 修饰方法**

```java
    public synchronized void fun() {
        System.out.println("同步方法");
    }
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

字节码信息如下。synchronized 修饰的方法并没有 monitorenter 指令和 monitorexit 指令，取得代之是 **ACC_SYNCHRONIZED** 标识，该标识指明了该方法是一个同步方法，JVM 通过该 ACC_SYNCHRONIZED 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。

![img](https://img-blog.csdnimg.cn/20200501194211368.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



# 锁

## **几种锁**

在JDK6及其以后，一个对象有四种锁状态，它们的级别由低到高依次是：

1. 无锁状态
2. 偏向锁状态
3. 轻量级锁状态
4. 重量级锁状态

其中的无锁就是没有对资源进行锁定，任何线程都可以尝试修改资源。其他的锁会随着竞争情况逐渐升级，锁的升级很容易发生，但是锁降级发生的条件会比较苛刻，锁降级发生在Stop The World期间，当JVM进入安全点的时候，会检查是否有闲置的锁，然后进行降级。

## **对象头**

Java的锁都是基于对象的，而每个对象都有对象头，若是非数组类型，则用2个字宽来存储对象头，如果是数组，则会用3个字宽来存储对象头。在32位处理器中，一个字宽是32位；在64位虚拟机中，一个字宽是64位。对象头的内容如下表：

| 长度     | 内容                   | 说明                         |
| -------- | ---------------------- | ---------------------------- |
| 32/64bit | Mark Word              | 存储对象的hashCode或锁信息等 |
| 32/64bit | Class Metadata Address | 存储到对象类型数据的指针     |
| 32/64bit | Array length           | 数组的长度（如果是数组）     |

其中的Mark Word格式如下。当对象状态为偏向锁时，Mark Word存储偏向的线程ID；当状态为轻量级锁时，Mark Word存储的是指向线程栈中Lock Record的指针；当状态为重量级锁时，Mark Word为指向堆中的monitor对象的指针。

| 锁状态   | 29 bit 或 61 bit             | 是否是偏向锁？             | 2锁标志位 |
| -------- | ---------------------------- | -------------------------- | --------- |
| 无锁     |                              | 0                          | 01        |
| 偏向锁   | 线程ID                       | 1                          | 01        |
| 轻量级锁 | 指向栈中锁记录的指针         | 此时这一位不用于标识偏向锁 | 00        |
| 重量级锁 | 指向互斥量（重量级锁）的指针 | 此时这一位不用于标识偏向锁 | 10        |
| GC标记   |                              | 此时这一位不用于标识偏向锁 | 11        |



## **偏向锁**

大多数情况下锁**不仅不存在多线程竞争，而且总是由同一线程多次获得**，因此便有了偏向锁。

偏向锁是指如果一段同步代码一直被一个线程所访问，那么该线程会自动获取锁，降低获取锁的代价。

**实现原理**

当一个线程第一次进入同步块时，会在对象头和栈帧中的锁记录里存储 锁的偏向线程ID，当下一次该线程进入这个同步块时，会去检查锁的Mark Word里面是不是放的自己的线程ID。：

（1）若是，则表明该线程已获得锁，以后该线程在进入和退出同步块时不需要花费CAS操作来加锁和解锁 ；

（2）若不是，则代表有另一个线程来竞争这个偏向锁。此时会尝试使用CAS来替换Mark Word里面的线程ID为新线程的ID，操作的结果分两种：

- 成功：表示之前的线程不存在了，那么Mark Word里面的线程ID为新线程的ID。此时的锁仍然为偏向锁；
- 失败：表示之前的线程仍然存在，那么暂停之前的线程，设置偏向锁标识为0，并设置锁标志位为00，升级为轻量级锁，接下来按照轻量级锁的方式进行竞争锁。

**撤销偏向锁**

当其他线程尝试竞争偏向锁时， 持有偏向锁的线程会释放锁，并重置偏向锁标识，大致过程如下：

1. 在一个安全的时间点（没有字节码正在执行）停止拥有锁的线程。
2. 遍历线程栈，如果存在锁记录的话，需要修复锁记录和Mark Word，使其变成无锁状态。
3. 唤醒被停止的线程，将当前锁升级成轻量级锁。

这个过程的开销很大，因此如果应用程序里所有的锁通常处于竞争状态，那么偏向锁就会是一种累赘，对于这种情况，我们可以一开始就把偏向锁这个默认功能给关闭：-XX:UseBiasedLocking=false。



## 轻量级锁

多个线程在不同时段获取同一把锁，这不存在锁竞争。因此，JVM采用轻量级锁来避免线程的阻塞和唤醒。

**加锁**

JVM首先将在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，用于存储锁对象目前的Mark Word的拷贝，然后拷贝对象头中的Mark Word复制到锁记录中。拷贝成功后，虚拟机将使用CAS操作尝试将对象的Mark Word更新为指向Lock Record的指针，并将Lock Record里的owner指针指向对象的Mark Word：

- 如果成功，当前线程获得锁，并且锁对象Mark Word的锁标志位设置为“00”，表示此对象处于轻量级锁定状态；
- 如果失败，表示Mark Word已经被替换成了其他线程的锁记录，说明在与其它线程竞争锁，当前线程就尝试使用自旋来获取锁。

自旋是指不断尝试获取锁，一般用循环来实现。

自旋是需要消耗CPU的，如果一直获取不到锁的话，那该线程就一直处在自旋状态，白白浪费CPU资源。因此JDK采用了适应性自旋的方式：如果线程自旋成功了，则下次自旋的次数会更多，如果自旋失败了，则自旋的次数就会减少。

如果自旋到一定程度，依然没有获取到锁，那么这个线程会阻塞。同时这个锁就会升级成重量级锁（或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁也会升级成重量级锁）。

**轻量级锁的释放**

在释放锁时，当前线程会使用CAS操作将Lock Record的内容复制回锁的Mark Word里面：

- 如果没有发生竞争，那么这个复制的操作会成功；
- 如果有其他线程因为自旋多次导致轻量级锁升级成了重量级锁，那么CAS操作会失败，此时会释放锁并唤醒被阻塞的线程。



## **重量级锁**

重量级锁依赖于操作系统的互斥量（mutex） 实现的，而操作系统中 线程间状态的转换需要相对比较长的时间，所以重量级锁效率很低，但被阻塞的线程不会消耗CPU。

当一个线程尝试获得锁时，如果该锁已经被占用，则会将该线程封装成一个ObjectWaiter对象插入到Contention List的队列的队首，然后调用park函数挂起当前线程。

当线程释放锁时，会从Contention List或EntryList中挑选一个线程x唤醒，x被唤醒后会尝试获取锁，但由于synchronized是非公平的，x不一定能获得锁。

如果线程获得锁后调用Object.wait方法，则会将线程加入到WaitSet中，当被Object.notify唤醒后，会将线程从WaitSet移动到Contention List或EntryList中去。需要注意的是，当调用一个锁对象的wait或notify方法时，如果当前锁的状态是偏向锁或轻量级锁，则会先膨胀成重量级锁。



## **锁升级流程的总结**

当一个线程准备获取共享资源时：

（1）检查对象锁的对象头里的MarkWord里面是不是放的自己的线程ID ：如果是，表示当前线程是处于 “偏向锁” ；若不是，通知持锁线程暂停，并把Markword的内容置为空。

（2）两个线程都把锁对象的HashCode复制到自己栈帧中的锁记录空间，接着开始通过CAS操作来把锁对象的MarKword的内容修改为自己锁记录空间的方式 来竞争MarkWord。

（3）上一步中成功执行CAS的获得资源，失败的则进入自旋。

（4）获得资源的线程执行完后，释放共享资源。如果自旋的线程在自旋过程中，成功获得资源，那么此时依然处于轻量级锁的状态；

（5）如果自旋失败达到一定次数 ，进入重量级锁的状态，这个时候，自旋的线程进行阻塞，等待之前线程执行完成并唤醒自己。



## **各种锁的优缺点**

| **锁**   | **优点**                                                     | **缺点**                                         | **适用场景**                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------ | ------------------------------------ |
| 偏向锁   | 加锁和解锁不需要额外的消耗，和执行非同步方法比仅存在纳秒级的差距。 | 如果线程间存在锁竞争，会带来额外的锁撤销的消耗。 | 适用于只有一个线程访问同步块场景。   |
| 轻量级锁 | 竞争的线程不会阻塞，提高了程序的响应速度。                   | 如果始终得不到锁竞争的线程使用自旋会消耗CPU。    | 追求响应时间。同步块执行速度非常快。 |
| 重量级锁 | 线程竞争不使用自旋，不会消耗CPU。                            | 线程阻塞，响应时间缓慢。                         | 追求吞吐量。同步块执行时间较长。     |



## **乐观锁与悲观锁**

锁可以从其他角度分类，而乐观锁和悲观锁是一种分类方式。

**悲观锁**

该锁认为每次访问共享资源时会发生冲突，所以必须对每次数据操作加上锁，以保证临界区的程序同一时间只能有一个线程在执行。

悲观锁多用于”写多读少“的环境，避免频繁失败和重试影响性能。

**乐观锁**

乐观锁总是假设对共享资源的访问没有冲突，线程可以不停地执行，无需加锁也无需等待。而一旦多个线程发生冲突，乐观锁通常是使用一种称为CAS的技术来保证线程执行的安全性。

由于无锁操作中没有锁的存在，因此不可能出现死锁的情况。乐观锁多用于“读多写少“的环境，避免频繁加锁影响性能



# **参考资料**

[深入浅出多线程](https://github.com/RedSpider1/concurrent)

[javaGuide](https://github.com/Snailclimb/JavaGuide)