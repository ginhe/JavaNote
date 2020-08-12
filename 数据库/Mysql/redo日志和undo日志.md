# redo日志

InnoDB存储引擎是以页为单位来管理存储空间，我们进行的增删改查实质都是在访问页面。在真正访问页面之前，需要把磁盘上的页缓存到内存中的Buffer Pool之后才可以访问。

持久性是事务的四个特性之一，该特性表示对于一个已经提交的事务，在事务提交后即使系统发生崩溃，这个事务对数据库所作的修改也不能丢失。如果我们只在Buffer Pool中修改了页面，假设事务提交后突然出现故障，导致内存中的数据失效，那么已提交的事务的修改也随之丢失了。那么我们又该如何保证这个持久性呢

我们可以**记录修改的哪些东西**，这样即使系统崩溃了，重启后按照记录的步骤更新数据页即可。记录这些修改步骤的内容称为redo日志。

执行语句的过程中产生的redo日志被划分为若干个不可分割的组，比如：

- 更新name属性产生的redo日志是不可分割的；
- 向聚簇索引对应B+树的页面中插入一条记录产生的redo日志是不可分割的。

以向某个索引对应B+树的页面中插入一条记录为例，在定位到这条记录需要插入到的数据页之后，可能有两种情况：

（1）该数据页的剩余空间足够容纳一条待插入记录，这种情况称为乐观插入。

（2）该数据页的剩余空间不足，则需要进行页分裂：新建一个叶子节点，把原数据页种的一部分复制到这个新数据页中，再把待插入记录添加进去，最后还要在内节点中添加一条目录项记录来指向这个新创建的页。这种情况称为悲观插入。这个步骤需要对多个页面进行修改，意味着会产生多条redo日志。

插入一条记录的过程必须是原子的，也就是说上面的所有步骤要么全部完成，要么没有执行。因此在执行这些需要原子性操作时，必须要以**组**的形式来记录redo日志。某个组中的redo日志，要么全部恢复，要么一条也不恢复。





### **Mini-Transaction的概念**





## redo日志 的组成

Redo日志分为：

- redo日志缓存redo log buffer，它保存在内存中，是易失的。
- redo日志文件redo log file，它保存在磁盘中，是持久的。



### redo日志缓存

通过mtr生成的redo日志放在一个页中，该页称为block。由于磁盘速度过慢，在写入redo日志时，不能直接写到磁盘。因此服务器在启动时就向OS申请了一片 被称为**redo日志缓存**的连续内存空间。每生成一条redo日志，就将其暂存到一个地方。等mtr结束后，将该过程产生的一组redo日志全部赋值到**redo日志缓存**中。

向**redo日志缓存**中写入redo日志的过程是顺序的：先往前边的block页写，写慢后往下一个block页写。

<img src="https://img-blog.csdnimg.cn/20200418220349368.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70" alt="img" style="zoom:50%;" />![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### redo日志文件

**redo日志缓存**中的日志默认刷新到MySQL数据目录下的ib_logfile0和ib_logfile1的⽂件中。

磁盘上的redo日志文件是以**一个日志文件组**的形式出现的，这些文件以ib_logfile[数字]的形式进行命名。在将redo日志写入日志文件组时，是从ib_logfile0开始写，若写满了则从下一个文件开始写。若最后一个文件写满了，则重新转到ib_logfile0继续写。

<img src="https://img-blog.csdnimg.cn/20200418201816899.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NjgxODMw,size_16,color_FFFFFF,t_70" alt="img" style="zoom:33%;" />![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)





## 何时写入redo

（1）在数据页修改完成后，脏页刷出磁盘之前写入redo日志。

（2）redo日志比数据页先写回磁盘

（3）聚集索引、二级索引、undo页面的修改，均需要记录Redo日志。



## redo的流程

**Mini-Transaction**

Mini-Transaction是指对底层页面中的一次原子访问过程，简称mtr。每个mini-transaction对应每一条DML操作，比如一条update语句，其由一个mini-transaction来保证。

**流程**

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4cb67f2150a54439b53a7c4465752731~tplv-k3u1fbpfcp-zoom-1.image" style="zoom: 50%;" />

以一个update语句为例：

- 将原始数据从磁盘读到内存中，修改数据的内存拷贝。
- 对数据修改后，生成一条重做日志（记录数据被修改后的值），并写入mini-transaction私有的Buffer中。
- update语句结束后，将日志拷贝到redo日志缓存。
- 当整个事务提交后，将redo日志缓存中的内容刷新到redo日志文件。对redo日志文件采用追加写的方式，一个文件写满后到下一个文件写。

> 一个redo日志文件组的容量是有限的，我们不得不循环使用redo日志文件组中的文件：如果redo日志已经写进磁盘，但对应的脏页仍停留在Buffer Pool，则redo日志占用的磁盘空间还无法被覆盖；只有当脏页刷新到磁盘里，redo日志占用的磁盘空间才可以被后续的redo日志重用。



## redo的刷盘时机

在一些情况下**redo日志缓存**里的一组redo日志会被刷新到磁盘里：

- **redo日志缓存**空间不足。当写入**redo日志缓存**的redo日志里已占了一半容量时，需要把这些日志刷新到磁盘上。
- 事务提交时。
- 后台有一个负责将**redo日志缓存**中的redo日志刷新到磁盘的的线程，大约每秒刷新一次。
- 正常关闭服务器时。



## redo如何保证事务的持久性

InnoDB存储引擎通过**Force Log at Commit 机制**实现事务的持久性：当事务提交时，先将redo日志缓存 写入 redo日志文件进行持久化，等到事务的提交操作完成时才算完成。在持久化一个数据页前，先将内存中相应的日志页持久化。

每次将redo日志缓存 写入redo日志文件时都会调用一次操作系统的fsync操作。这是因为MySQL工作在用户空间，其重做日志缓存处于用户空间的内存中，要写入到磁盘上的重做日志文件中，中间还需要经过操作系统内核空间的os buffer。调用fsync()的作用就是将os buffer中的日志刷到磁盘的重做日志文件中。





# undo日志

事务需要保证原子性，但有时事务执行到一半会出现一些情况：

- 事务执行过程中可能遇到各种错误，比如服务器本身的错误，操作系统的错误，突然断电导致的错误。
- 事务执行的过程中手动输入ROLLBACK语句结束当前事务的执行

这两种情况都会导致事务执行一半就结束，但在事务执行过程中可能已经修改了很多东西。为保证原子性，我们需要把东西改回原先的样子，这个过程称为回滚。为了回滚而记录的东西称为undo日志。

当执行ROLLBACK时，就可以从undo日志中的逻辑记录 读取到相应内容并回滚。所谓的回滚操作，实质做到是和之前相反的工作，比如一条INSERT ，对应一条 DELETE，对于每个UPDATE,对应一条相反的 UPDATE,将修改前的行放回去。

> undo日志还用于MVCC。



## undo日志写入时机

（1）DML操作修改聚簇索引前，记录undo日志

（2）二级索引记录的修改，不记录undo日志

（3）undo页面的修改，同样需要记录redo日志。





## undo日志是否是redo日志的逆过程

undo日志是逻辑日志，对事务回滚时，是将数据库逻辑地址恢复原样；

redo日志是物理日志，记录的是数据页的物理变化。

答案是否定的。



# 参考资料

《MySQL是怎样运行的：从根儿上理解MySQL》

[浅析MySQL事务中的redo与undo](https://juejin.im/post/5c3c5c0451882525487c498d)