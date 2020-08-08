# Executor框架

Executor框架可以控制线程的启动，执行和关闭通过 Executor 来启动线程比使用 Thread 的 start 方法更好，除了更易管理，效率更好（用线程池实现，节约开销）外，还有助于避免 this 逃逸问题。

在HotSpot VM的线程模型中，Java线程被一对一映射为本地操作系统线程，Java线程启动时会创建一个本地OS线程；当该Java线程终止时，这个OS线程也会被回收。OS会调度所有线程并将它们分配给可用的CPU。

在上层，Java多线程程序通常把应用分解为若干个任务，然后使用用户级的调度器（Executor框架）将这些任务映射为固定数量的线程；在底层，OS内核将这些线程映射到硬件处理器上。

![img](https://img-blog.csdnimg.cn/20200429085511739.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



## Executor框架的结构与成员

Executor是一个接口，它是Executor框架的基础，它负责任务的提交与任务的执行分离。

Executor框架主要由3大部分组成：

**任务**

包括被执行任务需要实现的接口：Runnable接口或Callable接口。

**任务的执行**

包括任务执行机制的核心接口Executor，以及继承自Executor的 **ExecutorService**接口。在Executor框架有两个关键类实现了ExecutorService接口 （ThreadPoolExecutor和ScheduledThreadPoolExecutor）。

- **ThreadPoolExecutor**是线程池的核心实现类，用来执行被提交的任务；

- **ScheduledThreadPoolExecutor**是一个实现类，可以在给定的延迟后运行命令，或者定期执行命令。ScheduledThreadPoolExecutor比Timer更灵活，功能更强大。

Runnable接口和Callable接口的实现类，都可以被ThreadPoolExecutor或ScheduledThreadPoolExecutor执行。

**异步计算的结果**

包括接口Future和实现Future接口的FutureTask类。

当我们把**Runnable接口**或**Callable接口**的实现类提交（调用submit方法）给**ThreadPoolExecutor**或**ScheduledThreadPoolExecutor**时，会返回一个**FutureTask对象**。



![img](https://img-blog.csdnimg.cn/20200429085657199.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



##  **Executor框架的使用步骤**

**（1）任务的创建**

主线程首先要创建实现Runnable或者Callable接口的任务对象，其中工具类Executors可以把一 个Runnable对象封装为一个Callable对象：

- Executors.callable(Runnable task)
- Executors.callable(Runnable task, Object resule)。

**（2）执行任务**

把任务对象交给ExecutorService执行：ExecutorService.execute(Runnable command)；

或者也可以把Runnable对象或Callable对象提交给ExecutorService执行：

- ExecutorService.submit(Runnable task)
- ExecutorService.submit(Callable task)

其中submit方法会返回一个实现Future接口的对象 ；

> 由于FutureTask实现了Runnable，我们也可以创建FutureTask，然后直接交给ExecutorService执行。

**（3）等待任务的执行**

主线程可以执行FutureTask.get()方法来等待任务执行完成。get方法会阻塞当前线程直到完成任务。

此外主线程可以执行 FutureTask.cancel（boolean mayInterruptIfRunning）来取消此任务的执行。



## ThreadPoolExecutor

线程池实现类ThreadPoolExecutor是JUC提供的一类线程池工具。

**为什么要使用线程池**

- 创建/销毁线程需要消耗系统资源，线程池可以**复用已创建的线程**。

- **控制并发的数量**。并发数量过多，可能会导致资源消耗过多，从而造成服务器崩溃。（主要原因）
- **对线程做统一管理**。

### 线程池构造参数

先来看一下ThreadPoolExecutor构造函数中的7个参数，我们先介绍前5个必须的参数，然后再介绍后两个非必须参数。

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) 
```

**（1）int corePoolSize**：**线程池中核心线程数最大值**。线程池中有两类线程，核心线程和非核心线程。核心线程默认情况下会一直存在于线程池中，即使这个核心线程什么都不干；而非核心线程如果长时间的闲置，就会被销毁。

（2）**int maximumPoolSize**：**该线程池中核心线程数量 + 非核心线程数量总数最大值** 。

（3）**long keepAliveTime**：**非核心线程闲置超时时长**。如果设置allowCoreThreadTimeOut(true)，则会也作用于核心线程。

（4）**TimeUnit unit**：keepAliveTime的单位。TimeUnit是一个枚举类型，它包括：微毫秒NANOSECONDS，微秒MICROSECONDS ，毫秒MILLISECONDS ，SECONDS，MINUTES ，HOURS ，DAYS 。

（5）**BlockingQueue workQueue**：阻塞队列，维护着**等待执行的Runnable任务对象**。

常用的几个阻塞队列：

1. **LinkedBlockingQueue**

   链式阻塞队列。底层数据结构是链表，默认大小是`Integer.MAX_VALUE`，也可以指定大小。

2. **ArrayBlockingQueue**

   数组阻塞队列。底层数据结构是数组，需要指定队列的大小。

3. **SynchronousQueue**

   同步队列。内部容量为0，每个put操作必须等待一个take操作，反之亦然。

4. **DelayQueue**

   延迟队列。只有当其指定的延迟时间到了，才能够从队列中获取到该元素 。

（6）**ThreadFactory threadFactory**：创建线程的工厂 ，用于批量创建线程，统一在创建线程时设置一些参数：比如是否守护线程、线程的优先级等。如果不指定，会新建一个默认的线程工厂。

（7）**RejectedExecutionHandler handler**：拒绝处理策略，线程数量大于最大线程数就会采用拒绝处理策略，四种拒绝处理的策略为 ：

1. **ThreadPoolExecutor.AbortPolicy**：默认拒绝处理策略，丢弃任务并抛出RejectedExecutionException异常。
2. **ThreadPoolExecutor.DiscardPolicy**：丢弃新来的任务，但是不抛出异常。
3. **ThreadPoolExecutor.DiscardOldestPolicy**：丢弃队列头部（最旧的）的任务，然后重新尝试执行程序（如果失败，重复此过程）。
4. **ThreadPoolExecutor.CallerRunsPolicy**：由调用线程处理该任务。



### 线程池的状态

线程池本身有一个调度线程，这个线程就是用于管理布控整个线程池里的各种任务和事务，例如创建线程、销毁线程、任务队列管理、线程队列管理等等。因此线程池也有自己的状态。`ThreadPoolExecutor`类中定义了一个`volatile int`变量**runState**来表示线程池的状态 ，分别为：

（1）线程池创建后处于**RUNNING**状态。

（2）调用**shutdown()**方法后处于**SHUTDOWN**状态，线程池不能接受新的任务，但能处理已添加的任务。

（3）调用**shutdownNow()**方法后处于**STOP**状态，线程池不能接受新的任务，中断所有线程，阻塞队列中没有被执行的任务全部丢弃。此时，poolsize=0,阻塞队列的size也为0。

（4）当所有的任务已终止，控制状态的属性ctl记录的”任务数量”为0，线程池会变为**TIDYING**状态。接着会执行terminated()函数

（5）线程池处在TIDYING状态时，**执行完terminated()方法之后**，就变成**TERMINATED**状态。此时线程池彻底终止。



### 处理任务流程

#### execute方法

处理任务的核心方法是execute，下面是JDK1.8的execute方法源码：

```java
//ctl包含两个参数：线程状态runState；激活的线程数workerCount 
//它的低29位用于存放当前的线程数，因此一个线程池在理论上最大的线程数是(2^29)-1；高3位是用于表示当前线程池的状态
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));

public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();   
    int c = ctl.get();
    // 1.工作线程数小于核心线程数corePoolSize,则调用addWorker创建核心线程执行任务
    if (workerCountOf(c) < corePoolSize) {
       if (addWorker(command, true))
           return;
       c = ctl.get();
    }
    // 2.如果工作线程数大于等于核心线程数，线程池的状态是RUNNING，并且可以添加进队列
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();		//2.1再次检查线程池状态
        // 2.1 如果线程池不是RUNNING状态，则remove这个任务，然后执行拒绝策略。
        if (! isRunning(recheck) && remove(command))
            reject(command);
            // 2.2 如果remove失败，则判断工作线程数是否为0：若为0，则创建一个非核心线程
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 如果这时创建非核心线程失败(当前线程总数不小于maximumPoolSize时)，就会执行拒绝策略。
    else if (!addWorker(command, false))
         reject(command);
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**为什么要二次检查线程池的状态?**

在多线程的环境下，线程池的状态是时刻发生变化的。很有可能刚获取线程池状态后线程池状态就改变了。判断是否将`command`加入`workqueue`是线程池之前的状态。倘若没有二次检查，万一线程池处于非**RUNNING**状态（在多线程环境下很有可能发生），那么`command`永远不会执行。



**总结一下处理流程**：

1. 如果工作线程数小于核心线程数，则创建核心线程数
2. 如果工作线程数大于等于核心线程数时，会尝试将任务添加进队列：
   1. 如果成功，则对线程池状态进行二次验证：只要是`RUNNING`的状态，就一定要保证有工作线程还在；如果不是`RUNNING`状态，则将任务从队列里删除。
      1. 如果移除失败，则会判断工作线程是否为0 ，如果为0 就创建一个非核心线程；
      2. 如果成功移除，则执行拒绝策略。此时线程池已不可用



#### addWorker方法

下面来看一下**addWorker**方法的上半该部分。它主要判断线程数量是否超出阔值，若超出则返回false，否则使用CAS增加工作线程数量

```java
// ThreadPoolExecutor.addWorker方法源码上半部分
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

         //判断线程池的是否可以接收新任务
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
             // 如果方法参数core是ture,表明需要创建的线程为核心线程，则先判断当前线程是否大于核心线程
             // 如果core是false,证明需要创建的是非核心线程，则先判断当前线程数是否大于总线程数
             // 如果不小于，则返回false
            if (wc >= CAPACITY || wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
             //使用cas增加工作线程数
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  
            //如果添加失败，并且线程池状态发生了改变，那就重来一遍
            if (runStateOf(c) != rs)
                continue retry;
 
        }
    }
```

**addWorker**方法源码的下半部分如下。该部分创建worker对象并初始化，然后启动这个对象。

```java
 // ThreadPoolExecutor.addWorker方法源码下半部分
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        // 1.创建一个worker对象
        w = new Worker(firstTask);
        // 2.实例化一个Thread对象
        final Thread t = w.thread;
        if (t != null) {
            // 3.线程池全局锁
            final ReentrantLock mainLock = this.mainLock;
            //锁住
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    //如果线程已经启动，则抛出异常
                    if (t.isAlive()) 
                        throw new IllegalThreadStateException();
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                // 如果添加成功，则启动这个线程
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        //启动线程失败,回滚
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

#### Worker类

`Worker`类实现了`Runnable`接口，它也是一个线程任务，来看一下Woker类的部分源码

```java
// Worker类部分源码
private final class Worker extends AbstractQueuedSynchronizer implements Runnable{
    final Thread thread;
    Runnable firstTask;

    Worker(Runnable firstTask) {
        setState(-1); // inhibit interrupts until runWorker
        this.firstTask = firstTask;
        this.thread = getThreadFactory().newThread(this);
    }
	//Woker类的run方法调用的是runWorker方法，该线程的任务就是自己。
    public void run() {
          runWorker(this);
    }

	
    //首先去执行创建worker前已有的任务.
    //执行完任务后，worker会在while循环中不断调用getTask方法从阻塞队列中获取任务然后调用task.run()执行任务,从而达到复用线程的目的。
    //只要getTask方法不返回null,此线程就不会退出。
	//线程在取出阻塞队列中的任务前需要判断线程池的状态，如果是STOP或者TERMINATED，返回null。

    final void runWorker(Worker w) {
        Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
        // 1.线程启动之后，通过unlock方法释放锁
        w.unlock(); // allow interrupts
        boolean completedAbruptly = true;
        try {
            // 2.Worker执行firstTask或从workQueue中获取任务，如果getTask方法不返回null,循环不退出
            while (task != null || (task = getTask()) != null) {
                // 2.1进行加锁操作，保证thread不被其他线程中断（除非线程池被中断）
                w.lock();
                // 2.2检查线程池状态，倘若线程池处于中断状态，当前线程将中断。 
                if ((runStateAtLeast(ctl.get(), STOP) ||
                     (Thread.interrupted() &&
                      runStateAtLeast(ctl.get(), STOP))) &&
                    !wt.isInterrupted())
                    wt.interrupt();
                try {
                    // 2.3执行beforeExecute 
                    beforeExecute(wt, task);
                    Throwable thrown = null;
                    try {
                        // 2.4执行任务
                        task.run();
                    } catch (RuntimeException x) {
                        thrown = x; throw x;
                    } catch (Error x) {
                        thrown = x; throw x;
                    } catch (Throwable x) {
                        thrown = x; throw new Error(x);
                    } finally {
                        // 2.5执行afterExecute方法 
                        afterExecute(task, thrown);
                    }
                } finally {
                    task = null;
                    w.completedTasks++;
                    // 2.6解锁操作
                    w.unlock();
                }
            }
            completedAbruptly = false;
        } finally {
            processWorkerExit(w, completedAbruptly);
        }
    }

}
```



### 如何创建线程池

（1）你可以通过构造方法实现。不过官方API文档并不推荐。

![](https://user-gold-cdn.xitu.io/2020/7/18/17361defab9e0ccd?w=852&h=163&f=png&s=148562)

（2）通过Executor 框架的工具类Executors来实现 

![](https://user-gold-cdn.xitu.io/2020/7/18/17361dfe4a47ecb6?w=874&h=297&f=png&s=293481)

我们可以创建三种类型的ThreadPoolExecutor：

- **FixedThreadPool**
- **SingleThreadExecutor**
- **CachedThreadPool**



#### newFixedThreadPool

它是一个可重用固定数量线程的线程池。适用于为满足资源管理，而需要限制当前线程数量的应用场景。如负载较重的服务器。

```java
public static ExecutorService newFixedThreadPool(int nThreads)
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory)
```

来看一下构造方法的实现：

```java
/*
	核心线程数量和总线程数量都是传入的参数nThreads，所以只能创建核心线程，不能创建非核心线程。
	LinkedBlockingQueue的默认大小是Integer.MAX_VALUE，因此线程池中的线程数不会超过corePoolSize
	执行过程：
		（1）如果当前运行的线程数小于corePoolSize，则创建新的线程来执行任务；
		（2）当前运行的线程数等于corePoolSize后，将任务加入LinkedBlockingQueue；
		（3）线程执行完（1）的任务后，会循环中反复从LinkedBlockingQueue中获取任务来执行；
*/
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
}

public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>(),
                                      threadFactory);
}
```



#### newSingleThreadExecutor

它是使用单个worker线程的线程池，适用于需要保证顺序地执行各个任务，并且在任意时间点，不会有多个线程活动的应用场景。

```java
/*
	通过方法传入的threadFactory在需要时创建线程
	构造方法的corePoolSize和maximumPoolSize参数值都被设置为1，其他参数和FixedThreadPool相同。
	执行流程：
		（1）若当前运行的线程数小于1，则创建一个新线程执行任务。
		（2）若当前线程池中有一个运行的线程后，将任务加入LinkedBlockingQueue
		（3）线程执行完（1）的任务后，会循环中反复从LinkedBlockingQueue中获取任务来执行；
*/
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory));
    }

public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
}
```



#### newCachedThreadPool

它是一个创建一个会根据需要而创建新线程的线程池，适用于执行很多的短期异步任务的小程序，或是负载较轻的服务器。

```java
/*
核心线程数为0，线程池总线程数为Integer.MAX_VALUE。阻塞队列是内部容量为0的同步队列SynchronousQueue

运行流程为：

（1）提交任务进线程池

（2）由于corePoolSize为0的关系，不创建核心线程，线程池最大为Integer.MAX_VALUE。

（3）试将任务添加到SynchronousQueue队列。

（4）如果SynchronousQueue入列成功，等待被当前运行的线程空闲后拉取执行。如果当前没有空闲线程，
那么就创建一个非核心线程，然后从SynchronousQueue拉取任务并在当前线程执行。

（5）如果SynchronousQueue已有任务在等待，入列操作将会阻塞。

*/

public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

当需要执行很多**短时间**的任务时，CacheThreadPool的线程复用率比较高， 会显著的**提高性能**。而且线程60s后会回收，意味着即使没有任务进来，CacheThreadPool并不会占用很多资源。

**FixedThreadPool与CachedThreadPool的区别**：

（1）因为 corePoolSize == maximumPoolSize ，所以FixedThreadPool只会创建核心线程。 而CachedThreadPool因为corePoolSize=0，所以只会创建非核心线程。

（2）在 getTask() 方法，如果队列里没有任务可取，线程会一直阻塞在 LinkedBlockingQueue.take() ，线程不会被回收。 而CachedThreadPool会在60s后收回。

（3）由于线程不会被回收，会一直卡在阻塞，所以**没有任务的情况下， FixedThreadPool占用资源更多**。

（4）都几乎不会触发拒绝策略，但是原理不同。FixedThreadPool是因为阻塞队列可以很大（最大为Integer最大值），故几乎不会触发拒绝策略；CachedThreadPool是因为线程池很大（最大为Integer最大值），几乎不会导致线程数量大于最大线程数，故几乎不会触发拒绝策略。



### 示例

任务类

```java
public class SimpleTask implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " Start. Time = " + System.currentTimeMillis());
        try {
            Thread.sleep(3000);     //模拟任务
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " End. Time = " + System.currentTimeMillis());

    }
}
```

测试类

```java
    @Test
    public void  testExecute() {
        ExecutorService pool = Executors.newFixedThreadPool(3);
        for(int i = 0; i < 6; i++) {
            Runnable task = new SimpleTask();
            pool.execute(task);
        }
        pool.shutdown();
        while (!pool.isTerminated()) {
        }
        System.out.println("finish");
    }

/*
可能的输出：
	pool-1-thread-1 Start. Time = 1595077467814
    pool-1-thread-2 Start. Time = 1595077467847
    pool-1-thread-3 Start. Time = 1595077467850
    pool-1-thread-1 End. Time = 1595077470815
    pool-1-thread-1 Start. Time = 1595077470815
    pool-1-thread-2 End. Time = 1595077470847
    pool-1-thread-2 Start. Time = 1595077470847
    pool-1-thread-3 End. Time = 1595077470850
    pool-1-thread-3 Start. Time = 1595077470850
    pool-1-thread-1 End. Time = 1595077473815
    pool-1-thread-2 End. Time = 1595077473848
    pool-1-thread-3 End. Time = 1595077473851
    finish
*/
```







## ScheduledThreadPoolExecutor

ScheduledThreadPoolExecutor继承`ThreadPoolExecutor`，并实现ScheduledExecutorService。它主要用于在给定的延迟之后运行任务，或者定期执行任务。

**ScheduledThreadPoolExecutor主要用来在给定的延迟后运行任务，或者定期执行任务。**

```java
public interface ScheduledExecutorService extends ExecutorService {

    public ScheduledFuture<?> schedule(Runnable command,long delay, TimeUnit unit);

    public <V> ScheduledFuture<V> schedule(Callable<V> callable,long delay, TimeUnit unit);

// 该方法在initialDelay时长后第一次执行任务，以后每隔period时长，再次执行任务。
// period是从任务开始执行算起的。
    public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                                  long initialDelay,
                                                  long period,
                                                  TimeUnit unit);
// 该方法在initialDelay时长后第一次执行任务，以后每当任务执行完成后，等待delay时长，再次执行任务。  
    public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                     long initialDelay,
                                                     long delay,
                                                     TimeUnit unit);
}
```

ScheduledThreadPoolExecutor使用的任务队列DelayQueue封装了一个PriorityQueue，PriorityQueue会对队列中的任务进行排序，执行所需时间短的放在前面先被执行(ScheduledFutureTask的time变量小的先执行)，如果执行所需时间相同则先提交的任务将被先执行(ScheduledFutureTask的squenceNumber变量小的先执行)。

当调用ScheduledThreadPoolExecutor的 **scheduleAtFixedRate()** 方法或者**scheduleWirhFixedDelay()** 方法时，会向ScheduledThreadPoolExecutor的 **DelayQueue** 添加一个实现了 **RunnableScheduledFutur** 接口的 **ScheduledFutureTask**



### 执行步骤



<img src="https://user-gold-cdn.xitu.io/2020/7/18/173621d4f3f03b53?w=1162&amp;h=794&amp;f=png&amp;s=175887" style="zoom: 33%;" />

（1）线程1从DelayQueue中获取已到期的ScheduledFutureTask（DelayQueue.take()）。到期任务是指ScheduledFutureTask的time大于等于当前系统的时间；

（2）线程1执行这个ScheduledFutureTask；

（3）线程1修改ScheduledFutureTask的time变量为下次将要被执行的时间；

（4）线程1把这个修改time之后的ScheduledFutureTask放回DelayQueue中（DelayQueue.add())。





### 执行方法

#### schedule方法

```java
 	@Test
    public void  testExecute() throws InterruptedException {
        ScheduledExecutorService pool = Executors.newScheduledThreadPool(5);
        System.out.println("现在的时间是：" + new Date());
        for(int i = 0; i < 3; i++) {
            Thread.sleep(1000);
            SimpleTask task = new SimpleTask();
            //延迟10s后执行任务task。任务只执行1次
            pool.schedule(task, 10, TimeUnit.SECONDS);
        }
        Thread.sleep(30000);
        System.out.println("现在的时间是 "+new Date());
        //关闭线程池
        pool.shutdown();
        while(!pool.isTerminated()){
            //等待所有任务完成
        }
        System.out.println("Finished all threads");

    }
/*
可能的输出：
	现在的时间是：Sat Jul 18 21:35:08 CST 2020
    pool-1-thread-1 Start. Time = 1595079319367
    pool-1-thread-2 Start. Time = 1595079320367
    pool-1-thread-3 Start. Time = 1595079321367
    pool-1-thread-1 End. Time = 1595079322367
    pool-1-thread-2 End. Time = 1595079323368
    pool-1-thread-3 End. Time = 1595079324368
    现在的时间是 Sat Jul 18 21:35:41 CST 2020
    Finished all threads
*/
```



#### scheduleAtFixedRate方法

scheduleAtFixedRate(Runnable command,long initialDelay,long period,TimeUnit unit)方法

该方法在初始延迟initialDelay后运行，然后在每个period执行任务。period是从两个任务的开始时间算起

```java
for (int i = 0; i < 3; i++) {
	Thread.sleep(1000);
	SimpleTask task = new SimpleTask();
	pool.scheduleAtFixedRate(task, 0, 10, TimeUnit.SECONDS);
}
```



#### scheduleWithFixedDelay方法

scheduleWithFixedDelay(Runnable command,long initialDelay, long delay,TimeUnit unit)方法

该方法在initialDelay时长后第一次执行任务，以后每当任务执行完成后，等待delay时长，再次执行任务。 delay时常是一个任务结束到下一个任务开始的时间段

```java
for (int i = 0; i < 3; i++) {
	Thread.sleep(1000);
	SimpleTask task = new SimpleTask();
	pool.scheduleWithFixedDelay(task, 0, 1, TimeUnit.SECONDS);
}
```







# 参考资料

[深入浅出java多线程](http://concurrent.redspider.group/article/03/12.html)

[JavaGuide](https://juejin.im/post/5b0f69e46fb9a009f41479b4)