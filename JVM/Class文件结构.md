## 前序

JVM可以理解的代码叫字节码（扩展名为.class的文件），它只面向虚拟机。Java虚拟机不与包括Java语言在内的任何程序语言绑定，它只与“class”文件 这种二进制文件格式所关联。例如，使用Java编译器把Java代码编译为Class文件，使用JRuby等其他语言可以通过其他编译器把他们的源程序编译成Class文件。因此Java程序无需重新编译即可在多种不同操作系统中运行。



------

## Class文件结构

Class文件是一组以8个字节为基础单位的二进制流，当遇到需要占用8个字节以上空间的数据项时，则会按照高位在前的方式分割成若干个8字节进行存储。

Class文件格式采用了“无符号数”和“表”来存储数据：

- 无符号数属于基本的数据类型，以u1，u2，u4，u8来分别代表1个字节、2个字节、4个字节和8个字节的无符号数。无符号数可以用来描述数组，索引引用，数量值或按照UTF-8编码构成字符串。
- 表是由多个无符号数或其他表作为数据项构成的复合数据类型。

无论是无符号数还是表，当需要描述同一类型但数量不定的多个数据时，经常会使用一个前置的容量计数器和若干个连续的数据项，此时我们称这一系列连续的某一类型的数据为某一类型的“集合”。

![img](https://img-blog.csdnimg.cn/20200424105200607.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 魔数与Class文件的版本

每个Class文件的头4个字节称为 魔数，它的唯一作用是确定这个文件是否为一个能被虚拟机接收的Class文件。使用魔数而不是扩展名来进行识别主要是基于安全考虑，因为文件扩展名可以随意改动。

紧接着魔数的4个字节存储的是Class文件的版本号：第5和第6个字节是此版本号。第7和第8个字节是主版本号。高版本的JDK能向下兼容以前版本的Class文件，但不能兼容之后版本的Class文件。

### 常量池

紧接着主，次版本号之后是常量池入口，它是Class文件结构中与其他项目管理最多的数据，通常也是占用Class文件空间最大的数据项目之一。此外，它是Class文件中第一个出现的表类型数据项目。

由于常量池中常量的数量是不固定的，所以在常量池的入口放置一项u2类型的数据，代表常量池容量计数值，计数从1开始的。

> 将第0项空出是因为在特定情况下，后面某些 指向常量池的索引值的数据 需要表示“不引用任何一个常量池”的含义，于是可以把索引值设置为0来表示。

常量池主要存放两大常量：字面量和符号引用 

（1）字面量比较接近Java语言层面的常量概念，如文本字符串，final常量值等。

（2）符号引用属于编译原理方面的概念，主要包括下面几类常量：

- 被模块导出或者开放的包
- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符
- 方法句柄和方法类型
- 动态调用点和动态常量

在虚拟机加载Class文件时，进行的是动态连接，即在Class文件中不会保持各个方法，字段最终在内存中的布局信息，如果这些字段，方法的符号引用不经过虚拟机在运行期转换的话，是无法得到真正的内存入口地址，也就无法直接被虚拟机使用。

当虚拟机做类加载时，将会从常量池获得对应的符号引用，在类创建或运行时解析，翻译到具体的内存地址之中。

常量池中的每一项常量都是一个表，如下图所示，这17张表都有一个共同特点：表结构起始的第一位是u1类型的标志位，代表当前常量属于哪种常量类型。例如标志位是0x07的常量是属于CONSTANT_Class_info类型，此类型的常量代表一个类或接口的符号引用。

![img](https://img-blog.csdnimg.cn/20200402141005209.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 访问标志

在常量池结束后，紧接着的2个字节代表访问标志。它用于识别一些类或者接口层次的访问信息，包括：这个Class是类还是接口；是否定义为public类型；是否定义为abstract类型；若是类，是否被声明为final；等等。

![img](https://img-blog.csdnimg.cn/20200402143040963.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 类索引，父类索引和接口索引集合

类索引，父类索引都是一个u2类型的数据，而接口索引集合是一组u2类型的数据的集合。Class文件由这三项数据来确定该类型的继承关系：类索引 用于确定这个类的全限定名，父类索引 用于确定这个类的父类的全限定名，接口索引集合用来描述这个类实现了哪些接口。

类索引和父类索引 用两个u2类型的索引值表示，它们各自指向一个类型为CONSTANT_Class_info的类描述符常量，通过CONSTANT_Class_info的类型的常量中的索引值可以找到定义在CONSTANT_Utf8_info类型的常量中的 全限定名字符串。下图是类索引查找过程：

![img](https://img-blog.csdnimg.cn/20200402144305903.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

对于接口索引集合，入口的第一项u2类型的数据为接口技术器，表示索引表的容量，若该类没有实现任何接口，则计数器值为0，后面接口的索引表不再占用任何字节。

> **全限定名**：例如Object类，它在源文件的全限定名是java.lang.Object ，而在Class文件的全限定名是java/lang/Object

### 字段表的集合

描述接口或类中声明的变量。Java语言中的“字段”包括类级变量和实例级变量，但不包括在方法内部声明的局部变量。

![img](https://img-blog.csdnimg.cn/20200402145416674.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

字段修饰符放在access_flags项目中，它是一个u2的数据类型，可以设置的标志位及其含义如下

![img](https://img-blog.csdnimg.cn/20200402145509686.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

字段表结构中还有两项索引值：name_index和descriptor_index。它们都是对常量池项的引用，分别代表字段的简单名称以及字段和方法的描述符。

> 简单名称：没有类型和参数修饰的方法或字段名称。
>
> 描述符：描述字段的数据类型，方法的参数列表和返回值。如下图所示：
>
> ![img](https://img-blog.csdnimg.cn/20200402145956563.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 方法表集合

方法表的结构和字段表一样，包括访问标志（access_flags）、名称索引（name_index）、描述符索引（descriptor_index）、属性表 集合（attributes）几项。

![img](https://img-blog.csdnimg.cn/20200402150232407.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

访问标志如下：

![img](https://img-blog.csdnimg.cn/20200402150411976.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

在Java语言中，要重载一个方法，除了要与原方法具有相同的简单名称之外，还要求必须拥有一个与原方法不同的特征前面。特征签名是指一个方法中各个参数在常量池中的字段符号引用的集合，由于返回值不包含在特征前面中，因此Java语言无法仅仅依靠返回值的不同来对一个方法重载。但在Class文件格式中，若两个方法有相同的名称和特征签名，但返回值不同，它们也是可以共存在同一个Class文件中的。



### 属性表集合

Class文件，字段表，方法表都可以携带自己的属性表集合，以描述某些场景专有的信息。

> 方法的定义可以通过方法表集合中的访问标志、名称索引、描述符索引来 表达清楚，而方法里的Java代码，经过Javac编译器编译成字节码指令后，存放在方法属性表集合中的一个名为"Code"的属性里。



## 参考资料

《深入理解Java虚拟机》

[JavaGuide](https://github.com/Snailclimb/JavaGuide#jvm)