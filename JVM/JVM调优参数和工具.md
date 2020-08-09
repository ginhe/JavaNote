# 调优

所有线程共享数据区为：新生代大小+老年代大小+持久代大小。如果在Java堆中增大新生代，则会减小老年代大小。

> 持久代：用于存放静态文件，如Java类的定义信息，方法等。持久代对垃圾回收没有显著影响，但有些应用可能会动态生成或调用一些类，因此需要调整最大堆内存和最小堆内存设置一个比较大的持久代空间来存放运行过程中动态增加的类型。

## 相关参数

### 调整最大堆内存和最小堆内存

**-Xmx， –Xms**：指定Java堆最大值和初始Java堆最小值。

默认空余堆内存小于40%时，JVM会增大堆直至达到-Xmx的最大限制；默认空余堆内存大于70%时，JVM会减少堆直至达到-Xms的最小限制。

开发过程中通常将这两个参数设置相同的值，目的是为了能在Java回收清理完堆后，不再分隔计算堆区的大小而导致浪费资源。

### 调整新生代和老年代的比值

**-XX:NewRatio**：新生代（eden+2*Survivor）和老年代的比值。官方推荐新生代占Java堆的3/8。

### 调整Survivor区和Eden区的比值

**-XX:SurvivorRatio**：设置两个Survivor区和eden的比值

### 设置年轻代和老年代的大小

**XX:NewSize** --- 设置年轻代大小

**-XX:MaxNewSize** --- 设置年轻代最大值

最好是官方的8:1:1的Eden和Survivor比例



## GC调优

（1）**将新对象预留在新生代**：由于Full GC 的成本远高于 Minor GC，因此需要尽可能将对象分配在新生代。实际项目中根据 GC 日志分析新生代空间大小分配是否合理，适当通过“-Xmn”命令调节新生代大小，最大限度降低新对象直接进入老年代的情况。

（2）**大对象进入老年代**：虽然大部分情况下，将对象分配在新生代是合理的。但是如果大对象首次在新生代分配，则可能出现空间不足导致很多年龄不够的小对象被分。因此可以设置大对象直接进入老年代。通过-XX:PretenureSizeThreshold 来设置直接进入老年代的对象大小。

（3）**合理设置进入老年代对象的年龄**：`-XX:MaxTenuringThreshold` 设置对象进入老年代的年龄大小，减少老年代的内存占用，降低 full gc 发生的频率。

（4）**设置稳定的堆大小**

若满足下面条件，则不需要进行GC优化：

- MinorGC 执行时间不到50ms；
- Minor GC 执行不频繁，约10s一次；
- Full GC 执行时间不到1s；
- Full GC 执行频率不算频繁，不低于10分钟1次。



# 工具

## JDK 命令行

### jps

列出正在运行的虚拟机进程，并显示虚拟机执行（main()函数所在主类）名称 以及 这些进程的本地虚拟机唯一ID。

命令格式为。

> jps [ options ] [ hostid ]

其中选项option代表用户希望查询的虚拟机信息，主要分为三类：类加载，垃圾收集，运行期编译状况，常用的option选项如下

![img](https://img-blog.csdnimg.cn/20200402090159798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

如下图所示，210636是本地虚拟机唯一ID。

![img](https://img-blog.csdnimg.cn/20200402082516233.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

（1）jps -l 输出主类的全名

![img](https://img-blog.csdnimg.cn/20200402082837779.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

（2）jps -v 输出虚拟机进程启动时JVM参数

（3）jps -m 输出传递给Java进程main函数的参数



### jstat

监视虚拟机各种运行状态信息。它可以显示本地或远程虚拟机进程中的类加载，内存，垃圾收集，即使编译等运行时数据。

命令格式如下。参数interval和count代表查询间隔和次数，若省略则只查询一次，比如 jstat -gc 2764 250 20 意思为每250毫秒查询一次进程2764垃圾收集状况，一共查询20次

> ```html
> jstat [ option vmid [interval[s|ms] [count]] ]
> ```
>
> ![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

如果是本地虚拟机进程，vmid与lvmid是一致的；若是远程虚拟机进程，则vmid的格式应为：

> [protocol:][//]lvmid[@hostname[:port]/servername]

**测试：**监视一台服务器的内存状况：

> jstat -gcutil 2764
>
> S0   S1   E    O    P      YGC YGCT   FGC FGCT  GCT
>
> 0.00 0.00 6.20  41.42  47.20   16   0.105    3   0.472  0.577

服务器的新生代Eden使用了6.2%的空间，2个Survivor区（S0，S1）里面为空，老年代Old和永久代Permanent分别使用了41/42%和47.20%的空间。程序运行以来共发生Minor GC（YGC，表示Young GC）16次，总耗时0.105秒；发生Full GC（FGC，表示Full GC）3次，总耗时（FGCT，表示Full GC Time）为0.472秒；所有GC总耗时（GCT，表示GC Time）为0.577秒。



### jinfo

实时查看和调增虚拟机各项参数。前面提到的jps命令的-v参数可以查看虚拟机启动时显示指定的参数列表，但如果想知道未被显示指定的参数的系统默认值，可以使用jinfo的 -flag进行查询。

**测试：**如果我们希望查看虚拟机是否使用了某个收集器，如下所示。结果是+表示使用的是CMS收集器；若冒号后面跟的是-，则表示不是使用给定的收集器

> jinfo -flag UseConcMarkSweepGC 210636
>
> -XX:+UseConcMarkSweepGC

**测试：**查询CMSInitiatingOccupancyFraction参数值

> jinfo -flag  CMSInitiatingOccupancyFraction 210636
> -XX:CMSInitiatingOccupancyFraction=-1



### jmap

生成堆存储快照。同时也可以查询finalize执行队列，Java堆和方法区的信息（如空间使用率，当前使用的是哪种收集器等）。

**测试：**在F盘生成a.bin堆存储快照（注：须在管理员模式下cmd）。可通过Visual 等工具分析文件

> jmap -dump:format=b,file=F:\a.bin  210636
>
> Dumping heap to F:\a.bin ...
> Heap dump file created



### jhat

与jmap搭配使用，用于分析其生成的堆转储快照。jhat内置一个微型的HTTP/Web服务器，使用户可以在浏览器查看分析结果。

**测试：**在http://localhost:7000/中查看前面生成的堆存储快照

> jhat f:\a.bin
>
> ...省略前面输出
>
> Snapshot resolved.
> Started HTTP server on port 7000
> Server is ready.





### jstack

生成虚拟机当前时刻的线程快照（当前虚拟机内每一条线程正在执行的方法堆栈的集合）。生成线程快照的目的是定位线程出现长时间听到的原因，如线程间死锁，死循环等。

**测试：**下面是一段死锁代码

```
public class JavaVM {
    private static Object resource1 = new Object();
    private static Object resource2 = new Object();

    public static void main(String[] args) throws Exception {
        new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (resource1) {
                    System.out.println(Thread.currentThread() + "get resource1");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread() + "waiting get resource2");
                    synchronized (resource2) {
                        System.out.println(Thread.currentThread() + "get resource2");
                    }
                }
            }
        }, "线程 1").start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread() + "waiting get resource1");
                    synchronized (resource1) {
                        System.out.println(Thread.currentThread() + "get resource1");
                    }
                }
            }
        }, "线程 2").start();
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

通过监视器jstack命令分析：

> jstack 148688
>
> Found one Java-level deadlock:
> =============================
> "线程 2":
>  waiting to lock monitor 0x0000000002bfaf98 (object 0x000000076bc2a1b8, a java.lang.Object),
>  which is held by "线程 1"
> "线程 1":
>  waiting to lock monitor 0x0000000002bf8708 (object 0x000000076bc2a1c8, a java.lang.Object),
>  which is held by "线程 2"
>
> Java stack information for the threads listed above:
> ===================================================
> "??程 2":
>     at JavaVM$2.run(JavaVM.java:47)
>     \- waiting to lock <0x000000076bc2a1b8> (a java.lang.Object)
>     \- locked <0x000000076bc2a1c8> (a java.lang.Object)
>     at java.lang.Thread.run(Thread.java:745)
> "线程 1":
>     at JavaVM$1.run(JavaVM.java:29)
>     \- waiting to lock <0x000000076bc2a1c8> (a java.lang.Object)
>     \- locked <0x000000076bc2a1b8> (a java.lang.Object)
>     at java.lang.Thread.run(Thread.java:745)
>
> Found 1 deadlock.



------

## JDK 可视化分析工具

### JConsole：Java监视与管理控制台

它的主要功能是通过JMX的MBean对系统进行信息收集和参数动态调整。

通过JDK/bin目录下的jconsole.exe启动JConsole后，会自动搜索出本机运行的所有虚拟机进程。它可以对远程虚拟机进行监控

![img](https://img-blog.csdnimg.cn/202004021007268.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://img-blog.csdnimg.cn/20200402101316339.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

​                                             **内存监控**

jconsole可以显示当前内存的详细信息，包括堆内存，非堆内存，新生代，survior区等使用情况。

点击 “执行CG”按钮可以强制执行一个Full GC

![img](https://img-blog.csdnimg.cn/20200402103825969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



​                                             **线程监控**

同样，jconsole可以做一次类似可视化的jstack命令的线程监控

![img](https://img-blog.csdnimg.cn/20200402104153137.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### 多合-故障处理工具VisualVM 

它是一个运行监视和故障处理的程序，可以做到：

- 显示虚拟机进程以及进程的配置、环境信息（jps、jinfo）。
- 监视应用程序的 CPU、GC、堆、方法区以及线程的信息（jstat、jstack）。
- dump 以及分析堆转储快照（jmap、jhat）。
- 方法级的程序运行性能分析，找到被调用最多、运行时间最长的方法。