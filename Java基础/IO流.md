# IO流概述
## 什么是io流
IO即Input/Output，也就是输入输出。程序和运行时数据是在内存中驻留，由CPU这个超快的计算核心来执行。在涉及到数据交换的地方，通常是磁盘、网络等，就需要IO接口。比如说在浏览器中访问某一页面，则浏览器就需要通过网络IO获取页面，

Java把所有的传统的流类型都放到在java io包下，用于实现输入和输出功能。
## io流分类
**（1）按照流的流向，IO流可以分为输入流和输出流**；

此处所说的输入/输出是从程序运行所在的内存的角度来划分的。如下图15.1所示，从程序运行所在的内存角度来考虑，该数据流为输出流；再比如图15.2，数据是从服务器的内存输出到网络，然后再流向客户端，因此服务器程序使用输出流，客户端使用输入流。


![](https://user-gold-cdn.xitu.io/2020/5/2/171d439287e36abf?w=763&h=142&f=png&s=61096)

**（2）按照操作单元划分，可以划分为字节流和字符流**。

字节流和字符流的用法几乎完全一样，不同字节流操作的数据单元是8位字节，而字符流操作的是16位字符。

> ##### 既然有了字节流,为什么还要有字符流?
>
> 不管是文件读写还是网络发送接收，信息的最小存储单元都是字节，但字节流处理**多个字节表示的东西**的时候有可能会出现乱码的问题，比如汉字，用字节流读取的时候有可能因为一位字节没有读到就变成了乱码。但字符流可以解决这样的问题：**字节流和编码表的组合就是字符流**。有了编码表可以确定这个汉字有多少个字节，这样字节流就可以根据位数准确的读写汉字了。

**（3） 按照流的角色划分为节点流和处理流**。

节点流是指可以从/向一个特定的节点（例如IO设备：磁盘，网络等）读/写数据的流；

处理流是指对一个已存在的流的连接和封装，通过所封装的流的功能调用实现数据读写。处理流的构造方法总是要带一个其他的流对象做参数

## IO流的原理
java的许多IO流类都是从4个抽象类派生出来的：

* InputStream/Reader: 所有的输入流的父类，前者是字节输入流，后者是字符输入流。
* OutputStream/Writer: 所有输出流的父类，前者是字节输出流，后者是字符输出流。

对于输入流InputStream和Reader而言，它们把输入设备抽象成一个“水管”。输入流使用隐式的记录指针来表示当前准备从哪个“水滴”开始读取。每当程序取出一个/多个“水滴”，记录指针后移。

![](https://user-gold-cdn.xitu.io/2020/5/2/171d447266f89fa1?w=618&h=183&f=png&s=65352)

对于输出流OutputStream和Writer而言，它们同样把输出设备抽象成一个”水管“，只是这个水管里面没有任何水滴，同样的，输出流页使用采用隐示指针来标识当前水滴即将放入的位置。当执行输出时，程序依次把“水滴”放入输出流的水管,指针后移。

![](https://user-gold-cdn.xitu.io/2020/5/2/171d4495040e93c6?w=568&h=222&f=png&s=54986)

## 流的分类

![](https://user-gold-cdn.xitu.io/2020/5/2/171d44c8f62904d3?w=1069&h=320&f=png&s=26897)


# 常用的io流用法
## InputStream和Reader
InputStream和Reader是所有输入流的抽象父类，本身不能创建实例来执行输入，但它们的方法是所有输入流都可使用的方法。

**InputStream的3个方法**

（1）int read()

从输入流中读取单个字节，返回所读取的字节数据。

（2）int read(byte[]b)

从输入流中最多读取b.length个字节的数据，并将其存储在字节数组b中，返回实际读取的字节数。

（3）int read(byte[] b,int off,int len)

从输入流中最多读取len个字节的数据，并从off位置为起点，存储在数组b中，然后返回实际读取的字节数。

**Reader的3个方法**

（1）int read()

从输入流中读取单个字符，返回所读取的字符数据。

（2）int read(char[]b)

从输入流中最多读取b.length个字符的数据，并将其存储在字符数组b中，返回实际读取的字符数。

（3）int read(byte[] b,int off,int len)

从输入流中最多读取len个字符的数据，并从off位置为起点，存储在数组b中，然后返回实际读取的字符数。

**InputStream和Reader提供的一些移动指针的方法**：

void mark(int readAheadLimit);
在记录指针当前位置记录一个标记（mark）。

boolean markSupported(); 判断此输入流是否支持mark()操作，即是否支持记录标记。

void reset(); 将此流的记录指针重新定位到上一次记录标记（mark）的位置。

long skip(long n); 记录指针向前移动n个字节/字符。


## OutputStream和Writer
OutputStream和Writerd 的用法也类似，都提供如下三个方法：

（1）void write(int c)

将指定的字节/字符输出到输出流中，其中c可以代表字节或字符。


（2）void write(byte[]/char[] buf);

将字节数组/字符数组中的数据输出到指定输出流中。

（3）void write(byte[]/char[] buf, int off,int len );

将字节数组/字符数组中从off位置开始，长度为len的字节/字符输出到输出流。


此外对于Writer来说，可以用字符串代替字符数组作为参数：

（4）void write(String str);

将str字符串里包含的字符输出到指定输出流中。


（5）void write (String str, int off, int len);

将str字符串里面从off位置开始，长度为len的字符输出到指定输出流中。
****
## 文件流的使用
**输入流FileInputStream / FileReader负责读入Test.txt文件内容**

```java
public class FileRead {
    public static void main(String[] args) throws IOException{
        FileInputStream fis = null;     //FileReader fis=null;
        try {
            fis = new FileInputStream("F:\\Test.txt");
            byte[] b = new byte[1024];  //char[] b = new char[1024]
            int curRead = 0;
            while ((curRead = fis.read(b)) > 0) {
                System.out.println(new String(b, 0, curRead));
            }
        }catch (IOException e){
            e.printStackTrace();
        }finally {
            fis.close();
        }
    }
}
```

**输出流 FileOutputStream把原有的Test文件内容读写到新建的newTest文件**
```java
public class FileOutput {
    public static void main(String[] args) throws IOException {
        FileOutputStream fos = null;    
        FileInputStream fis = null;
        try { //把原有的Test文件内容读写到新建的newTest文件中
            fos = new FileOutputStream("F:\\newTest.txt");
            fis = new FileInputStream("F:\\Test.txt");

            byte[] b = new byte[1024];
            int curRead = 0;
            while((curRead = fis.read(b)) > 0) {
                fos.write(b, 0, curRead);
            }
        }catch (IOException e){
            e.printStackTrace();
        }finally {
            fis.close();
            fos.close();
        }
    }
}
```


## 缓冲流的使用

**为什么使用缓冲流**

对于不带缓冲的操作，每写入一个字节/字符就要写入一个字节/字符，但由于涉及磁盘的IO操作要比内存的操作慢得多，因此不带缓冲的流效率很低；对于带缓冲的流，就会一次性读取很多字节/流，但不向磁盘写入，而是先放入内部缓存区数组，等达到了缓冲区大小后一次性写入磁盘，这样就可以减少磁盘操作次数。

缓冲字节流**BufferedInputStream**和**BufferedOutputStream**分别是**FilterInputStream**和**FilterOutputStream**的子类；

缓冲字符流**BufferedReader**和**BufferedWriter**分别是**Reader**和**Writer**的子类。

**缓冲流的示例**

同样是通过输出流把原有的文件写入到新建的一个文件。
```java
public class Test {
    public static void main(String[] args)throws IOException  {
        BufferedInputStream bis = null;
        BufferedOutputStream bos = null;
        try {
            //原有的文件Test.txt
            bis = new BufferedInputStream(new FileInputStream("F:\\Test.txt"));
            bos = new BufferedOutputStream(new FileOutputStream("F:\\newTest.txt"));
            byte[] b = new byte[1024];
            int curRead = 0;
            while ((curRead = bis.read(b)) > 0) {
                bos.write(b, 0, curRead);
            }
            bos.flush();  //写入磁盘
        }
        catch (IOException e){
        	e.printStackTrace();
        }finally {
            bis.close();
            bos.close();
        }
    }
}
```

##  转换流的使用
以获取键盘输入为例：Java使用System.in表示键盘输入，这个输入流是字节流InputStream类的实例，如果我们键盘输入的内容都是文本（更适合使用字符流），因此可以使用InputStreamReader将其包装成BufferedReader。


```java
public class Test {
    public static void main(String[] args)throws IOException  {

        try {
            //将字节输入流System.in转化为字符输入流
            InputStreamReader reader = new InputStreamReader(System.in);
            BufferedReader br = new BufferedReader(reader);
            String curRead = null;
            //BufferedReader可以一次读取一行文本。
            while ((curRead = br.readLine()) != null) {
                if(curRead.equals("exit")) {
                    System.exit(1);
                }
                System.out.println("键盘输入内容：" + curRead);
            }
        }
        catch (IOException e){
        	e.printStackTrace();
        }finally {
        }
    }
}
```

## 对象流的使用

我们可以通过对象流和文件流 把一个类对象写成一个本地文件，不过前提条件是这个类和类的成员类型必须实现序列化接口。

首先实现一个Person类


```java
public class Person implements Serializable {
    //如果不希望一些属性被序列化，可以在前面加上transient关键字
    private int age;
    private String name;

    public Person(String name, int age) {
        this.age = age;
        this.name = name;
    }

    public Person() {
    }
    //省略get/set方法
    @Override
    public String toString() {
        return "Person{" +
                "age=" + age +
                ", name='" + name + '\'' +
                '}';
    }
}

```

然后是相关的实现类：

```java
public class ObjectIO {
    //将obj对象保存到filePath目录下
    public static void saveObject(Object obj, String filePath) {
        File file = new File(filePath);
        FileOutputStream fos = null;
        ObjectOutputStream oos = null;
        try {
            fos = new FileOutputStream(file);
            oos = new ObjectOutputStream(fos);
            oos.writeObject(obj);
            oos.flush();

            oos.close();
            fos.close();
        }catch (Exception e) {
            e.printStackTrace();
        }
    }
    //读取filePath目录下的对象文件。
    public static <T> T readObject(String filePath) {
        File file = new File(filePath);
        Object obj = null;
        try {
            FileInputStream fis = new FileInputStream(file);
            ObjectInputStream ois = new ObjectInputStream(fis);
            obj = ois.readObject();

            ois.close();
            fis.close();
        }catch (Exception e) {
            e.printStackTrace();
        }
        return (T)obj;
    }
}

```

最后测试一些即可：


```java
public class Test {
    public static void main(String[] args)throws IOException  {
        Person person = new Person("abc", 13);

        ObjectIO.saveObject(person, "F:\\person.txt");
        Person readPerson = ObjectIO.readObject("F:\\person.txt");
        System.out.println(readPerson);
    }
}
```


# BIO、NIO、AIO

首先Java中的IO都是依赖操作系统内核进行的，我们程序中的IO读写其实调用的是操作系统内核中的read&write两大系统调用。

那内核是如何进行IO交互的呢？

1. 网卡收到经过网线传来的网络数据，并将网络数据写到内存中。
2. 当网卡把数据写入到内存后，网卡向cpu发出一个中断信号，操作系统便能得知有新数据到来，再通过网卡中断程序去处理数据。
3. 将内存中的网络数据写入到对应socket的接收缓冲区中。
4. 当接收缓冲区的数据写好之后，应用程序开始进行数据处理。

对应抽象到java的socket代码如下所示：

```java
 // 监听指定的端口
    int port = 8080;
    ServerSocket server = new ServerSocket(port);
    // server将一直等待连接的到来
    Socket socket = server.accept();
    // 建立好连接后，从socket中获取输入流，并建立缓冲区进行读取
    InputStream inputStream = socket.getInputStream();
    byte[] bytes = new byte[1024];
    int len;
    while ((len = inputStream.read(bytes)) != -1) {
      //获取数据进行处理
      String message = new String(bytes, 0, len,"UTF-8");
    }
    // socket、server，流关闭操作，省略
  }
```

这段代码和底层内核的网络IO很类似，主要体现在accept()等待从网络中的请求到来然后bytes[]数组作为缓冲区等待数据填满后进行处理。而BIO、NIO、AIO之间的区别就在于这些操作是同步还是异步，阻塞还是非阻塞。

**什么是同步和异步**

同步和异步指的是一个执行流程中每个方法是否必须依赖前一个方法完成后才可以继续执行。假设我们的执行流程中：依次是方法一和方法二。

- 同步：调用一旦开始，调用者必须等到方法调用返回后，才能继续后续的行为。即方法二一定要等到方法一执行完成后才可以执行。

- 异步：调用立刻返回，调用者不必等待方法内的代码执行结束。即执行方法一的时候，直接交给其他线程执行，不由主线程执行，也就不会阻塞主线程。

也就是说，同步与异步关注的是方法的执行方是主线程还是其他线程。

**什么是阻塞和非阻塞**

阻塞与非阻塞指的是单个线程内遇到同步等待时，是否在原地不做任何操作。

- 阻塞：遇到同步等待后，一直在原地等待同步方法处理完成。

- 非阻塞：遇到同步等待，不在原地等待，先去做其他的操作，隔断时间再来观察同步方法是否完成。





## BIO  (Blocking I/O)

它是JDK1.4之前的传统IO模型，本身是同步阻塞I/O模式。

如果是在单线程环境下：一般通过在while(true) 循环中调用 accept() 方法等待接收客户端的连接的方式监听请求，接收请求后建立套接字socket，并在socket上进行读写操作，此时不能再接收其他客户端连接请求（socket.accept()、socket.read()、socket.write() 涉及的三个主要函数都是同步阻塞的）。

如果是在多线程环境下：通常有一个独立的 Acceptor线程负责监听客户端的连接，它接收到客户端的连接请求后，为每个客户端创建一个新线程进行链路处理。处理完后，通过输出流返回应答给客户端，销毁线程。
![](https://user-gold-cdn.xitu.io/2020/5/3/171d7dea501b493f?w=734&h=485&f=png&s=59771)

该模式存在一个问题：线程的创建和销毁的成本都很高，如果并发访问量增加会导致线程急剧增加，则会导致线程堆栈溢出，创建新线程失败等问题，并最终导致进程宕机或者僵死，不能对外提供服务。


**代码示例**

客户端负责发送问候信息

```java
public class IOClient {
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Socket socket = new Socket("127.0.0.1", 3333);
                    while (true) {
                        try {
                            socket.getOutputStream().write((new Date() + "：halo").getBytes());
                            Thread.sleep(2000);
                        }catch (InterruptedException e)    {}
                    }
                }catch (IOException e) { }
            }
        }).start();
    }
}
```
服务端负责为每一个客户端的连接请求分配一个读取信息线程

```java
public class IOServer {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(3333);
        new Thread(new Runnable() {
            @Override
            public void run() {
                while(true) {
                    try {
                        Socket socket = serverSocket.accept();
                        new Thread(new Runnable() {         //为每一个连接分配一个读取线程
                            @Override
                            public void run() {
                                try {
                                    int curLen;
                                    byte[] data = new byte[1024];
                                    InputStream inputStream = socket.getInputStream();
                                    while ((curLen = inputStream.read(data)) != -1) {
                                        System.out.println(new String(data, 0, curLen));
                                    }
                                }catch (IOException e)  {}
                            }
                        }).start();
                    }catch (IOException e) { }
                }
            }
        }).start();

    }
}
```


### 伪异步IO
为了解决同步阻塞I/O面临的一个链路需要一个线程处理的问题，线程模型得到了优化：后端通过一个线程池来处理多个客户端的请求接入，形成客户端个数M：线程池最大线程数N的比例关系，其中M可以远远大于N。通过线程池可以灵活地调配线程资源，设置线程的最大值，防止由于海量并发接入导致线程耗尽。

![](https://user-gold-cdn.xitu.io/2020/5/3/171d7eaaa6c262e7?w=732&h=496&f=png&s=66389)
如上图所示，当有新的客户端接入时，将客户端的 Socket 封装成一个线程Task，然后放到线程池中处理。由于线程池可以设置消息队列的大小和最大线程数，因此，它的资源占用是可控的，无论多少个客户端并发访问，都不会导致资源的耗尽和宕机。

但是由于伪异步I/O通信框架的底层仍采用同步阻塞的BIO模型，当面对十万甚至百万级连接时，传统的 BIO 模型是无能为力的。

## NIO (New I/O)
NIO是一种同步非阻塞的I/O模型，它对应着JDK的java.nio包，它提供接口了Channel，抽象类Selector和Buffer等。

NIO提供了与传统BIO模型中的 Socket 和 ServerSocket 相对应的 SocketChannel 和 ServerSocketChannel两种不同套接字，**这两个套接字都支持阻塞和非阻塞两种模式**：阻塞模式简单但性能和可靠性不好，非阻塞模式则与之相反。
对于低并发的应用程序，可以使用同步阻塞I/O来提升开发速率和更好的维护性；对于高负载、高并发的（网络）应用，应使用 NIO 的非阻塞模式来开发。

### NIO与IO流的区别
**（1）IO流是阻塞的，NIO流是不阻塞的**

对于NIO而言，单线程可以在从通道读/写数据到buffer时，同时做其他事情，当数据读取到buffer后，线程再继续处理数据。

而IO流则是阻塞的，当一个线程调用 read() 或 write() 时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能做其他事情。


**（2）Buffer缓存区**

NIO库中加入的字节数组Buffer是一个包含一些要写入或者要读出的数据的对象，它是NIO与原IO的一个重要区别。

在面向流的I/O中，是将数据直接写入或者将数据直接读到 Stream 对象中，虽然 Stream 中也有 Buffer开头的扩展类，但那是从流读到缓冲区，而 NIO是直接读到 Buffer 中进行操作。

<img src="https://user-gold-cdn.xitu.io/2020/7/6/17322239896b02ae?w=932&amp;h=354&amp;f=png&amp;s=6821" style="zoom:50%;" />

在NIO厍中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的; 在写入数据时，写入到缓冲区中。任何时候访问NIO中的数据，都是通过缓冲区进行操作。

<img src="https://user-gold-cdn.xitu.io/2020/7/6/17322239ef88c51d?w=981&amp;h=335&amp;f=png&amp;s=6471" style="zoom:50%;" />

最常用的缓冲区是 ByteBuffer,一个 ByteBuffer 提供了一组功能用于操作 byte 数组。每一种Java基本类型（除了Boolean类型）都对应有一种缓冲区。

**（3）通道Channel **

NIO通过通道来读取和写入数据。

传统流的读写是单向的：即要么是输入，要么是输出；而通道是双向的：既可以写数据到*通道*，又可以从*通道*中读取数据

**（4）多用复用器Selector **


在传统的IO流中，每当一个连接来了，都会创建一个线程，对应一个用于不断检测连接是否有数据可读的while死循环，大多情况下1个while循环里同一时刻只有少量数据可读。

而NIO的选择器会不断的轮询注册在其上的 Channel，如果某个 Channel 上面有新的 TCP 连接接入、读和写事件，这个 Channel 就处于就绪状态，会被 Selector 轮询出来，然后通过 SelectionKey 可以获取就绪 Channel 的集合，进行后续的 I/O 操作。


![](https://user-gold-cdn.xitu.io/2020/5/3/171d810c65761272?w=368&h=292&f=png&s=8652)

总结来说，虽然NIO 编程难度确实比同步阻塞 BIO 大很多，但它的优点也很多：

1. 客户端发起的连接操作是异步的，在多用复用器 Selector等待后续结果，而不需要传统IO的客户端那样被同步阻塞。
2. SocketChannel 的读写操作都是异步的，如果没有可读写的数据它不会同步等待，直接返回，这样IO通信线程就可以处理其它的链路，不需要同步等待这个链路可用。
3. 由于 JDK 的 Selector 在 Linux 等主流操作系统上通过 epoll 实现，它没有连接句柄数的限制(只受限于操作系统的最大句柄数或者对单个进程的句柄限制)，这意味着一个 Selector 线程可以同时处理成千上万个客户端连接，而且性能不会随着客户端的增加而线性下降，因此，它非常适合做高性能、高负载的网络服务器。



### NIO 读数据和写数据方式

**读取**

创建一个缓存区，然后请求通道读取数据。

**写入**

创建一个缓冲区，填充数据，并要求通道写入数据。


## 代码示例

**服务端程序**
```java
public class NIOServer {
    public static void main(String[] args) throws IOException {
        //NIO模型通常有两个线程：server负责轮询是否有新的连接；client负责轮询连接是否有数据可读
        Selector server = Selector.open();
        Selector client = Selector.open();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    //服务端启动
                    ServerSocketChannel listen = ServerSocketChannel.open();
                    listen.socket().bind(new InetSocketAddress(3333));
                    listen.configureBlocking(false);
                    listen.register(server, SelectionKey.OP_ACCEPT);

                    while (true) {
                        // 监测是否有新的连接，这里的1指的是阻塞的时间为1ms
                        if(server.select(1) > 0) {
                            Set<SelectionKey> set = server.selectedKeys();
                            Iterator<SelectionKey> keyIterator = set.iterator();

                            while(keyIterator.hasNext()) {
                                SelectionKey key = keyIterator.next();
                                if(key.isAcceptable()) {
                                    try {
                                        //每新来一个连接，无需创建一个线程，而是直接注册到clientChannel
                                        SocketChannel clientChannel =
                                                ((ServerSocketChannel)key.channel()).accept();
                                        clientChannel.configureBlocking(false);
                                        clientChannel.register(client, SelectionKey.OP_READ);
                                    }finally {
                                        keyIterator.remove();
                                    }
                                }
                            }
                        }
                    }
                }catch (IOException ignored) {
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    while (true) {
                        //批量轮询是否有哪些连接有数据可读，这里的1指的是阻塞的时间为1ms
                        if(client.select(1) > 0) {
                            Set<SelectionKey> set = client.selectedKeys();
                            Iterator<SelectionKey> keyIterator = set.iterator();
                            while (keyIterator.hasNext()) {
                                SelectionKey key = keyIterator.next();

                                if (key.isReadable()) {
                                    try {
                                        SocketChannel clientChannel = (SocketChannel) key.channel();
                                        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                                        // (3) 面向 Buffer读取数据
                                        clientChannel.read(byteBuffer);
                                        byteBuffer.flip();
                                        System.out.println(
                                                Charset.defaultCharset().newDecoder().decode(byteBuffer).toString());
                                    } finally {
                                        keyIterator.remove();
                                        key.interestOps(SelectionKey.OP_READ);
                                    }
                                }

                            }
                        }
                    }
                }catch (IOException ignored) {
                }
            }
        }).start();
    }
}
```

JDK原生NIO的编程及其复杂，并且其底层由 epoll实现，该实现饱受诟病的空轮询 bug 会导致 cpu 飙升 100%。而Netty的出现改善了  原生NIO 出现的这些问题。



# 参考资料

[java IO体系的学习总结](https://blog.csdn.net/nightcurtis/article/details/51324105)

[JavaGuide](https://github.com/Snailclimb/JavaGuide)

[BIO，NIO](https://developer.aliyun.com/article/726698)

[如何理解NIO，BIO，AIO](https://juejin.im/post/6844903985158045703)