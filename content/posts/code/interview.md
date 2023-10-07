---
title: "面试题"
date: 2023-03-05T23:50:31+08:00
draft: false
Tags : ["面试", "Python"]
Categories : ["面试", "Python"]
---

# 面试题

### 线程是并发还是并行，进程是并发还是并行？

线程是并发，进程是并行；

进程之间互相独立，是系统分配资源的最小单位，同一个线程中的所有线程共享资源。

### 并行(parallel)和并发(concurrency)?

并行：同一时刻多个任务同时在运行。

并发：在同一时间间隔内多个任务都在运行，但是并不会在同一时刻同时运行，存在交替执行的情况。

实现并行的库有：multiprocessing

实现并发的库有：threading程序需要执行较多的读写，请求和回复任务的需要大量的IO操作，IO密集型操作使用并发更好。CPU运算量大的程序使用并行会更好。

### python GIL锁是什么？GIL锁的作用？说说你对GIL的理解？

GIL的全称是全局解释器锁，也就是一个解释器一个锁，GIL其实是功能和性能之间权衡后的产物，它有其存在的合理性，也有较难改变的客观因素。

限制多线程同时执行，保证同一时间只有一个线程执行，所以cpython里的多线程其实是伪多线程。GIL的目的是确保每个进程中只有一个线程运行，所以多个进程之间是不会互相影响的，多进程确实可以用来削弱GIL的负面影响，但是对于IO密集型操作事务，多进程理论上也会比多线程快一点，但是因为多进程消耗的资源也比多线程大很多，所以说你只有少数的任务并发，你用多进程没有问题，但是并发任务多的情况而且是IO密集型操作，用多线程就比多进程好得多，毕竟多线程占用资源少，比多进程更加便于管理，多进程的管理比多线程要复杂而且不确定。

### 谈谈你对多进程，多线程，以及协程的理解？

进程：一个运行的程序（代码）就是一个进程，没有运行的代码就程序，进程是系统资源分配的最小单位，进程拥有自己独立的内存空间，所以进程间数据不共享，开销大。

线程：调度执行的最小单位，也叫执行路径，不能独立存在，依赖进程存在一个进程至少有一个线程，叫主线程，而多个线程共享内存（数据共享，共享全局变量），从而极大地提高了程序的运行效率。

协程：是一种用户态的轻量级线程，协程的调度完全由用户控制。协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈，直接操作栈则基本没有内核切换的开销，可以不加锁的访问全局变量，所以上下文的切换非常快。

### TCP是怎么连接怎么退出

通过三次握手建立连接，通讯完成时四次挥手

### 简述TCP和UDP的区别以及优缺点？

UDP是面向无连接的通讯协议，UDP数据包括目的端口号和源端口号信息。

优点：UDP速度快，操作简单，要求系统资源较少，由于通讯不需要连接，可以实现广播发送

缺点：UDP传送数据前并不与对方建立连接，对接收到的数据也不发送确认信号，发送端不知道数据是否会正确接收，也不重复发送，不可靠。

TCP是面向连接的通讯协议，通过三次握手建立连接，通讯完成时四次挥手

优点：TCP在数据传递时，有确认、窗口、重传、阻塞等控制机制，能保证数据正确性，较为可靠。

缺点：TCP相对于UDP速度慢一点，要求系统资源较多。

### IO多路复的作用

socketserver，多个客户端连接，单线程下实现并发效果，就叫多路复用。与多进程和多线程相比，I/O多路复用技术的最大优势是系统开销小。系统不必创建进程/线程，也不必维护这些进程/线程，从而大大减小了系统的开销。

### python设计模式

单例，工厂，装饰器，生成器

**创建型模式，共五种：工厂方法模式、抽象工厂模式、单例模式、建造者模式、原型模式。**

**结构型模式，共七种：适配器模式、装饰器模式、代理模式、外观模式、桥接模式、组合模式、享元模式。**

**行为型模式，共十一种：策略模式、模板方法模式、观察者模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式。**

### python中的列表和元组有什么区别

列表是动态的，长度大小不固定，可以随意的增加、删除、修改元素

元组是静态的，长度在初始化的时候就已经确定不能更改，更无法增加，删除，修改元素

### 什么是python迭代器？

迭代器是可以遍历或迭代的对象

### python中的生成器是什么？

返回可迭代项集的函数称为生成器

### Redis基本类型

Redis支持五种数据类型：string（字符串）、hash（哈希）、list（列表）、set（集合）及zset（sortedset：有序集合）

### python垃圾回收机制

引用计数，标记清除，分代回收

1. 引用计数

​ 引用计数是用来记录对象被引用的次数，每当对象被创建或者被引用时将该对象的引用次数加一，当对象的引用被销毁时该对象的引用次数减一，当对象的引用次数减到零时，那么其作占用的空间也就可以被释放了。

2. 标记清除

​ 标记清除算法是一种基于追踪回收技术实现的垃圾回收算法。它分为两个阶段：第一阶段是标记阶段，第二阶段是把那些没有标记的对象非活动对象进行回收。

3. 分代回收

​ 将系统中的全部内存块依据其存活时间划分为不同的集合，每个集合就成为一个“代”，垃圾收集的频率随着“代”的存活时间的增大而减小。

### Python数据类型（前三个不可变类型，后三个可变类型）

整型、字符串、元组、集合、列表、字典

### 常用的Python标准库都有哪些？

os操作系统、time时间、random随机、pymysql连接数据库、threading线程、multiprocessing进程、queue队列

第三方库：

django和flask也是第三方库，requests、virtualenv、selenium、scrapy、xadmin、celery、re、hashlib、md5

常用的科学计算库（如Numpy、Scipy、Pandas）

### 简述Django的orm

ORM，全拼Object-Relation Mapping，意为对象-关系映射。实现了数据模型与数据库的解耦，通过简单的配置就可以轻松更换数据库，而不需要修改代码只需要面向对象编程，orm操作本质上会根据对接的数据库引擎，翻译成对应的sql语句，所有使用Django开发的项目无需关心程序底层使用的是MySQL、Oracle、sqlite...... 如果数据库迁移，只需要更换Django的数据库引擎即可。

### 对MVC、MVT解读的理解

MVC:

M: Model 模型 和数据库进行交互

V: View 视图 负责产生HTML页面

C: Controller 控制器 接收请求 进行处理 与M和V进行交互，返回应答

MVT:

M: Model 模型 和MVC中的M功能相同，和数据库进行交互

V: View 视图 和MVC中的C功能相同，接收请求，进行处理，与M和T进行交互，返回应答

T: Template 模板 和MVC中的V功能相同，产生HTML页面

### django请求的生命周期？

当用户数据url时，会生成请求头和请求体发给服务端

url经过django中的web服务器，再经过中间件，最后url到路由映射表匹配

匹配成功就执行对应的视图函数，视图函数根据客户端的请求查询相应的数据，返回给django，然后django把客户端想要的数据作为一个字符串返回给客户端

客户端接收到返回的数据，经过渲染后显示给用户

### MySQL常见数据库引擎及比较

InnoDB

支持事务

支持表锁 行锁（for update）

表锁：select * from tb for update

行锁：select id,name from tb where id=2 for update

myisam

查询速度快

全文索引

支持表锁

表锁：select * from tb for update

NDB

高可用、高性能、高可扩展性的数据库集群系统

Memory

默认使用的是哈希索引

### 什么是事务？MySQL如何支持事务？

事务用于将某些操作的多个SQL作为原子性操作，一旦有某一个出现错误，即可回滚到原来的状态，从而保证数据库数据完整性

事务的特性：

原子性：确保工作单元内的所有操作都成功完成，否则事务将被中止在故障点，和以前的操作将回滚到以前的状态

一致性：确保数据库正确地改变状态后，成功提交的事务

隔离性：使事务操作彼此独立的和透明的

持久性：确保提交的事务的结果或效果的系统出现故障的情况下仍然存在

Mysql实现事务

InnoDB支持事务，MyISAM不支持

启动事务

start transaction

update from account set money=money-100 where name="a";

update from account set money=money+100 where name="b";

commit;

'start transaction 手动开启事务， commit 手动关闭事务'

### MySQL索引类型

1. 普通索引：是最基本的索引，它没有任何限制
2. 唯一索引：与前面的普通索引类似，不同的就是：索引列的值必须唯一，但允许有空值
3. 主键索引：是一种特殊的唯一索引，一个表只能有一个主键，不允许有空值
4. 组合索引：指多个字段上创建的索引，只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用，使用组合索引时遵循最左前缀集合
5. 全文索引：主要用来查找文本中的关键字，而不是直接与索引中的值相比较。fulltext索引跟其他索引大不相同，它更像是一个搜索引擎，而不是简单的where语句的参数匹配

### 数据库优化方案

1. 优化索引、SQL语句、分析慢查询
2. 设计表的时候严格根据数据库的设计范式类设计数据库
3. 使用缓存，把经常访问到的数据而且不需要经常变化的数据放在缓存中，能节约磁盘IO
4. 优化硬件，采用SSD，使用磁盘队列技术（RAID0,RAID1,RDID5)等
5. 采用MySQL内部自带的表分区技术，把数据分层不同的文件，能够提高磁盘的读取效率
6. 垂直分表：把一些不经常读的数据放在一张表里，节约磁盘I/O
7. 主从分离读写：采用主从复制把数据库的读操作和写入操作分离开来
8. 分库分表分机器（数据量特别大），主要的原理就是数据路由
9. 选择合适的表引擎，参数上的优化
10. 进行架构级别的缓存，静态化和分布式
11. 不采用全文索引
12. 采用更快的存储方式，例如NoSQL存储经常访问的数据

### 事务的特性

1. 原子性（Atomicity）：事务中的全部操作在数据库中是不可分割的，要么全部完成，要么均不执行
2. 一致性（Consistency）：几个并行执行的事务，其执行结果必须与按某一顺序串行执行的结果相一致
3. 隔离性（Isolation）：事物的执行不受其他事物的干扰，事务执行的中间结果对其它事务必须是透明的。
4. 持久性（Durability）：对于任意已提交事务，系统必须保证该事务对数据库的改变不被丢失，即使数据库出现故障

### Mysql的四种隔离级别

Read Uncommitted（读取未提交内容）

在该隔离级别，所有事物都可以看到其它未提交事务的执行结果。本隔离级别很少用于实际应用，因为它的性能也不比其它级别好多少。读取未提交的数据。也被称之为脏读（Dirty Read）

Read Committed（读取提交内容）

这是大多数数据库系统的默认隔离级别（但不是MySQL默认的）。它满足了隔离的简单定义：一个事务只能看见已经提交事务所做的改变。这种隔离级别也支持所谓的不可重复读（Nonrepeatable Read），因为同一事务的其他实例在该实例处理期间可能会有新的commit，所以同一select可能返回不同结果。

Repeatable Read（可重复读）

这是MySQL的默认事务隔离级别，它确保同一事务的多个实例在并发读取数据时，会看到同样的数据行。不过理论上，这会导致另一个棘手的问题：幻读（Phantom Read）。简单的说，幻读指当用户读取某一范围的数据行时，另一个事务又在该范围内插入了新行，当用户再读取该范围的数据行时，会发现有新的"幻影"行。InnoDB和Falcon存储引擎通过多版本并发控制（MVCC, Multiversion Concurrency Control）机制解决了该问题

Serializable（可串行化）

这是最高的隔离级别，它通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁，在这个级别，可能导致大量的超时现象和锁竞争。

### 简述redis有哪几种持久化策略及比较

RDM：每隔一段时间对redis进行一次持久化

- 缺点：数据不完整
- 优点：速度快

AOF: 把所有命令保存起来，如果想到重新生成redis，那么就把命令重新执行一次

- 缺点：速度慢，文件比较大
- 优点：数据完整

### Redis主从同步

主从刚刚连接的时候，进行全{{}y同步；全同步结束后，进行增量同步

### 说说HTTP和HTTPS区别？

HTTP和HTTPS的区别主要如下：

1. https协议需要到ca申请证书，一般免费证书较少，因而需要一定费用
2. http是超文本传输协议，信息是明文传输，https则是具有安全性的ssl加密传输协议
3. http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443
4. http的连接很简单，是无状态的；https协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议 ，比http协议安全

### 什么是JWT

​ json web token(JWT) 是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准（RFC7519) 该token被设计为紧凑且安全的，特 别适用于分布式站点的单点登陆（SSO）场景。JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其他业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。

### JWT和Session方式存储ID的区别

Session方式存储用户id的最大弊病在于Session是存储在服务器端的，所以需要占用大量服务器内存，对于较大型应用而言可能还要保存许多的状态。一般而言，大型应用还需要借助一些KV数据库和一系列缓存机制来实现Session的存储。

而JWT方式将用户状态分散到了客户端中，可以明显减轻服务端的内存压力。除了用户id之外，还可以存储其他的和用户相关的信息，例如该用户是否是管理员，用户所在的分组等。虽说JWT方式让服务器有一些计算压力（例如加密、编码和解码），但是这些压力相比磁盘存储而言可能就不算什么了。具体是否采用，需要在不同场景下用数据说话。

### JWT的构成

第一部分我们称它为头部（header），第二部分我们称其为载荷（payload，类似于飞机上承载的物品），第三部分是签证（signature）

### 常用的数据结构

数组 堆栈 队列 链表 树 图 字典树 哈希表

### 什么是数据结构？

简单说，数据结构就是一个容器，以某种特定的布局存储数据。这个"布局"使得数据结构在某些操作上非常高效，在另一些操作上则不那么高效。你的目标就是理解数据结构，这样就能为手头的问题选择最优的数据结构。

### 你知道哪些排序算法

冒泡、选择、快排、归并

### 什么是MongoDB

MongoDB是一个文档数据库，提供好的性能，领先的非关系型数据库。采用BSON存储文档数据，用c++编写的

### MySQL与MongoDB之间最基本的差别是什么？

MySQL和MongoDB两者都是免费开源的数据库。MySQL和MongoDB有许多基本差别包括数据的表示（data representation）查询、关系、事务、schema的设计和定义，标准化（normalization）、速度和性能

通过比较MySQL和MongoDB，实际上我们是在比较关系型和非关系型数据库，即数据存储结构不同

### MongoDB成为最好NoSQL数据库的原因是什么？

- 面向文件的
- 高性能
- 高可用性
- 易扩展型
- 丰富的查询语言

### 为什么用MongoDB?

- 架构简单
- 没有复杂的连接
- 深度查询能力，MongoDB支持动态查询
- 容易调试
- 容易扩展
- 不需要转化/映射应用对象到数据库对象
- 使用内存作为存储工作区，以便更快的存取数据

### 在哪些场景使用MongoDB？

- 大数据
- 内容管理系统
- 移动端Apps
- 数据管理

### MongoDB中的分片是什么意思

分片是将数据水平切分到不同的物理节点。当应用数据越来越大的时候，数据量也会越来越大。当数据量增长时，单台机器有可能无法存储数据或可接受的读取写入吞吐量。利用分片技术可以添加更多的机器来应对数据量增加以及读写操作的要求

### MongoDB支持主键外键关系吗

默认MongoDB不支持主键和外键关系。用MongoDB本身的API需要硬编码才能实现外键关联，不够直观且难度较大

### MongoDB支持哪些数据类型

String Integer Double Boolean Object ID Arrays Min/Max Keys Datetime Code Regular Expression等

### 在MongoDB中什么是索引

索引用于高效的执行查询，没有索引的MongoDB将扫描整个集合中的所有文档，这种扫描效率很低，需要处理大量的数据

索引是一种特殊的数据结构，将一小块数据集合保存为容易遍历的形式，索引能够存储某种特殊字段或字段集的值，并按照索引指定的方式将字段值进行排序

### 如何添加索引

使用db.collection.createIndex()在集合中创建一个索引

### 索引类型有哪些？

单字段索引(Single Field Indexes)

复合索引(Compound Indexes)

多健索引(Multikey Indexes)

全文索引(text Indexes)

Hash 索引(Hash Indexes)

通配符索引(Wildcard Index)

2dsphere(2dsphere Indexes)

### 分析器在MongoDB中的作用是什么？

MongoDB中包括了一个可以显示数据库中每个操作性能特点的数据库分析器。通过这个分析器你可以找到比预期慢的查询(或写操作)；利用这一信息，比如，可以确定是否需要添加索引。

分片(sharding)和复制(replication)是怎样工作的？

分片可能是单一的服务器或者集群组成，推荐使用集群

数据在什么时候才会扩展到多个分片(shard)里？

MongoDB分片是基于区域的，所以一个集合的所有对象都放置在同一个块中，只有当存在多余一个块的时候，才会有多个分片获取数据的选项

### 如何执行事务/加锁？

MongoDB没有使用传统的锁或者复杂的带回滚的事务，因为它设计的宗旨是轻量，快速以及可预计的高性能。可以把它类比成MySQLMyISAM的自动提交模式。通过精简对事务的支持，性能得到了提升，特别是一个可能会穿过多个服务器的系统里。

### 我应该启动一个集群分片(sharded)还是一个非集群分片的MongoDB环境？

为开发便捷起见，我们建议以非集群分片(unsharded)方式开始一个MongoDB环境，除非一台服务器不足以存放你的初始数据集。从非集群分片升级到集群分片(sharding)是无缝的，所以在你的数据集还不是很大的时候没必要考虑集群分片(sharding)

### MongoDB支持存储过程吗？如果支持的话，怎么用？

MongoDB支持存储过程，它是JavaScript写的，保存在db.system.js表中

### 如果一个分片(shard)停止或很慢的时候，发起一个查询会怎样？

如果一个分片停止了，除非查询设置了"Partial"选项，否则查询会返回一个错误。如果一个分片响应很慢，MongoDB会等待它的响应

### 深拷贝和浅拷贝有什么区别？

在创建新实例类型时使用浅拷贝，并保留在新实例中复制的值。浅拷贝用于复制引用指针，就像复制值一样。这些引用指向原始对象，并且在类的任何成员中所做的更改也将影响它的原始副本。浅拷贝允许更快地执行程序，它取决于所使用的数据的大小。

深拷贝用于存储已复制的值。深拷贝不会将引用指针复制到对象。它引用一个对象，并存储一些其他对象指向的新对象。原始副本中所做的更改不会影响使用该对象的任何其他副本。由于为每个被调用的对象创建了某些副本，因此深拷贝会使程序的执行速度变慢。

### Django如何实现websocket？

django实现websocket官方推荐大家使用channels。channels通过升级http协议升级到websocket协议。保证实时通讯。也就是说，我们完全可以用channels实现我们的即时通讯。而不是使用长轮询和计时器方式来保证伪实时通讯。它通过改造django框架，使Django既支持http协议又支持websocket协议。

### Django和flask的区别

django走的是大而全的方向，开发效率高。它的MTV框架，自带ORM，admin后台管理，自带sqlite数据库和开发测试用的服务器，给开发者提高了开发效率。重量级的web框架，功能齐全，提高一站式的解决思路，能让开发者不用在选择上花费大量的时间。

自带ORM和模板引擎，支持jinja等非官方模板引擎。

自带ORM使Django和关系型数据库耦合度高，如果要使用非关系型数据库，需要使用第三方库。

自带数据库管理APP。

成熟稳定，开发效率高，相对于flask，Django的整体封闭性比较好，适合做企业级网站的开发。

python web框架的先驱，第三方库丰富。

Flask是轻量级的框架，自由灵活，可扩展性，核心基于Werkzeug WSGI工具和jinja2模板引擎

适合做小网站以及web服务的API，开发大型网站无压力，但是架构需要自己设计

与关系型数据库的结合不弱于Django，而与非关系型数据库的结合远远优于django。

> ### Python GIL锁是什么？GIL锁的作用？说说你对GIL的理解？

GIL的全称是全局解释器锁，也就是一个解释器一个锁，GIL其实是功能和性能之间权衡后的产物，它有其存在的合理性，也有较难改变的客观因素。

限制多线程同时执行，保证同一时间只有一个线程执行。所以cpython里的多线程其实是伪多线程

GIL的目的是确保每个进程中只有一个线程运行，所以多个进程之间是不会互相影响的，多进程确实可以用来削弱GIL的负面影响，但是对于IO密集型操作事务，多进程理论上也会比多线程快一点，但是因为多进程消耗的资源也比多线程大很多，所以说你只有少数的任务并发，你用多进程没有问题，但是并发任务多的情况而且是IO密集型操作，用多线程就比多进程好的多，毕竟多线程占用资源少，比多进程更加便于管理，多进程的管理比多线程要复杂而且不稳定。

> ### 解释一下什么是锁，有哪几种锁？

锁(Lock)是Python提供的对线程控制的对象。有互斥锁，可重入锁，死锁

> ### 什么是死锁？

若干子线程在系统资源竞争时，都在等待对方对某部分资源解除占用状态，结果是谁也不愿先解锁，互相干等着，程序无法执行下去，这就是死锁。

> ### 谈谈你对多进程 ，多线程，以及协程的理解?

```python
进程：一个运行的程序(代码)就是一个进程，没有运行的代码叫程序，进程是系统资源分配的最小单位，进程拥有自己独立的内存空间，所以进程间数据不共享，开销大。
线程：调度执行的最小单位，也叫执行路径，不能独立存在，依赖进程存在一个进程至少有一个线程，叫主线程，而多个线程共享内存（数据共享，共享全局变量），从而极大地提高了程序的运行效率。
协程：是一种用户态的轻量级线程，协程的调度完全由用户控制。协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈，直接操作栈则基本没有内核切换的开销，可以不加锁的访问全局变量，所以上下文的切换非常快
```

> ### Python中的进程与线程的使用场景？

多进程适合在CPU密集型操作(CPU操作指令比较多，如位数多的浮点运算)

多线程适合在IO密集型操作(读写数据操作较多的 ，比如爬虫)

> ### 线程是并发还是并行，进程是并发还是并行？

进程之间相互独立，是系统分配资源的最小单位，同一个线程中的所有线程共享资源

> ### 并行(parallel)和并发(concurrency)?

并行：同一时刻多个任务同时在运行

并发：在同一时间间隔内多个任务都在运行，但是并不会在同一时刻同时运行，存在交替执行的情况

实现并行的库有：multiprocessing

实现并发的库有：threading

程序需要执行较多的读写、请求和回复任务的需要大量的IO操作，IO密集型操作使用并发更好。

CPU运算量大的程序，使用并行会更好

> ### IO密集型和CPU密集型区别？

IO密集型：系统运作，大部分的状况是CPU在等I/O(硬盘/内存)的读/写。

CPU密集型：大部分时间用来做计算、逻辑判断等CPU

> ### TCP是怎么连接怎么退出

通过三次握手建立连接，通讯完成时四次挥手

> ### 简述TCP和UDP的区别以及优缺点？

UDP是面向无连接的通讯协议，UDP数据包括目的端口号和源端口号信息

优点：UDP速度快、操作简单、要求系统资源较少，由于通讯不需要连接，可以实现广播发送

缺点：UDP传送数据前并不与对方建立连接，对接收到的数据也不发送确认信号，发送端不知道数据是否会正确接收，也不重复发送，不可靠

TCP是面向连接的通讯协议，通过三次握手建立连接，通讯完成时四次挥手

优点：TCP在数据传递时，有确认、窗口、重传、阻塞等控制机制，能保证数据正确性，较为可靠。

缺点：TCP相对于UDP速度慢一点，要求系统资源较多

> ### IO多路复用的作用

socketserver 多个客户端连接，单线程下实现并发效果，就叫多路复用。与多进程和多进程技术相比，I/O多路复用技术的最大优势是系统开销小，系统不必创建进程/线程，也不必维护这些进程/线程，从而大大减少了系统的开销

> ### 什么是粘包？socket中造成粘包的原因是什么？哪些情况会发生粘包现象？

只有TCP有粘包现象，UDP永远不会粘包

粘包：在获取数据时，出现数据的内容不是本应该接收的数据，如：对方第一次发送hello，第二次发送world，我方接收时，应该接收两次，一次是hello，一次是World，但事实上是一次收到helloworld，一次收到空，这种现象叫粘包

原因：粘包问题主要还是因为接收方不知道消息之间的界限，不知道一次性提取多少字节的数据所造成的

什么情况会发生：

1. 发生端需要等缓冲区满才发送出去，造成粘包（发送数据时间间隔很短，数据量很小，会合到一起，产生粘包）
2. 接收方不及时接收缓冲区的包，造成多个包接收（客户端发送了一段数据，服务端只收了一小部分，服务端下次再收的时候还是从缓冲区拿上次遗留 的数据，产生粘包）

> ### HTTP协议状态码有什么用，列出你知道的http协议的状态码，然后讲出它们都表示什么意思？

通过状态码告诉客户端服务器的执行状态，以判断下一步该执行什么操作

常见的状态机器码有：

100-199：表示服务器成功接收部分请求，要求客户端继续提交其余请求才能完成整个处理过程

200-299：表示服务器成功接收请求并已完成处理过程，常用200（OK 请求成功）

300-399：为完成请求，客户需要进一步细化请求。302（所有请求页面已经临时转移到新的url）

304/307（使用缓存资源）

400-499：客户端请求有错误，常用404（服务器无法找到被请求页面），403（服务器拒绝访问，权限不够）

500-599：服务端出现错误，常用500（请求未完成，服务器遇到不可预知的情况）

> ### 说说HTTP和HTTPS区别？

HTTPS和HTTP的区别主要如下：

1. https协议需要到ca申请证书，一般免费证书较少，因而需要一定费用。
2. http是超文本传输协议，信息是明文传输，https则是具有安全性的ssl加密传输协议
3. http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443
4. http的连接很简单，是无状态的；https协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全

> ### 请简单说一下三次握手和四次挥手？

三次握手过程：

1. 首先客户端向服务端发送一个带有SYN标志，以及随机生成的序号100(0字节)的报文
2. 服务端收到报文后返回一个报文(SYN200(0字节)，ACK1001(字节+1))给客户端
3. 客户端再次发送带有ACk标志201(字节+)序号的报文给服务端

至此三次握手过程结束，客户端开始向服务端发送数据

1. 客户端向服务端发起请求：我想给你通信，你准备好了吗？
2. 服务端收到请求后回应客户端：I`ok，你准备好了吗？
3. 客户端礼貌的再次回一下服务端：准备就绪，咱们开始通信吧！

整个过程跟打电话的过程一模一样：1 喂，你在吗 2 在，我说的你听得到不  3 恩，听得到(接下来开始你的表演)

补充：SYN：请求询问  ACK：回复，回应

四次挥手过程：

由于TCP连接是可以双向通信的(全双工)，因此每个方向都必须单独进行关闭(这句话才是精辟，后面四个挥手过程都是其具体实现的语言描述)

四次挥手过程，客户端和服务端都可以先开始断开连接

1. 客户端发送带有fin标识的报文给服务端，请求通信关闭
2. 服务端收到信息后，回复ACK答应关闭客户端通信(连接)请求
3. 服务端发送带有fin标识的报文给客户端，也请求关闭通信
4. 客户端回应ACK给服务端，答应关闭服务端的通信(连接)请求

### 什么是MongoDB

MongoDB是一个文档数据库，提供好的性能，领先的非关系型数据库。采用BSON存储文档数据，用c++编写的

### MySQL与MongoDB之间最基本的差别是什么？

MySQL和MongoDB两者都是免费开源的数据库。MySQL和MongoDB有许多基本差别包括数据的表示(data representation), 查询，关系，事务，schema的设计和定义，标准化(normalization),速度和性能

通过比较MySQL和MongoDB，实际上我们是在比较关系型和非关系型数据库，即数据存储结构不同

### MongoDB成为最好NoSQL数据库的原因是什么？

- 面向文件的
- 高性能
- 高可用性
- 易扩展型
- 丰富的查询语言

### 为什么用MongoDB？

- 架构简单
- 没有复杂的连接
- 深度查询能力，MongoDB支持动态查询
- 容易调试
- 容易扩展
- 不需要转化/映射应用对象到数据库对象
- 使用内部内存作为存储工作区，以便更快的存取数据

### 在哪些场景使用MongoDB？

- 大数据
- 内容管理系统
- 移动端Apps
- 数据管理

### MongoDB中的分片是什么意思？

分片是将数据水平切分到不同的物理节点。当应用数据越来越大的时候，数据量也会越来越大。当数据量增长时，单台机器有可能无法存储数据或可接受的读取写入吞吐量。利用分片技术可以添加更多的机器来应对数据量增加以及读写操作的要求

### MongoDB支持主键外键关系吗？

默认MongoDB不支持主键和外借关系。用MongoDB本身的API需要硬编码才能实现外键关联，不够直观且难度较大

### MongoDB支持哪些数据类型

String Integer Double Boolean Object Object ID Arrays Min/Max Keys Datetime Code Regular Expression等

### 在MongoDB中什么是索引

索引用于高效的执行查询，没有索引的MongoDB将扫描整个集合中的所有文档，这种扫描效率很低，需要处理大量的数据。

索引是一种特殊的数据结构，将一小块数据集合保存为容易遍历的形式，索引能够存储某种特殊字段或字段集的值，并按照索引指定的方式将字段值进行排序

### 如何添加索引

使用db.collection.createIndex()在集合中创建一个索引

### 如何查询集合中的文档

db.collectionName.find({key:value})

### 用什么方法可以格式化输出结果

db.collectionName.find().pretty()

### 更新数据

db.collectionName.update({key:value},{$set:{newkey:newValue}})

### 如何删除文档

db.collectionName.remove({key: value})

### 在MongoDB中如何排序

使用1和-1来指定排序方式，其中1表示升序，而-1表示降序

db.connectionName.find({key:value}).sort({columnName:1})

### 什么是聚合

聚合操作能够处理数据记录并返回计算结果。聚合操作能将多个文档中的值组合起来，对成组数据执行各种操作，返回单一的结果。它相当于SQL中的count(*)组合group by。对于MongoDB中的聚合操作，应该使用aggregate()方法。

db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)

### 在MongoDB中什么是副本集(避免单点故障)

在MongoDB中副本集由一组MongoDB实例组成，包括一个主节点多个次节点，MongoDB客户端的所有数据都写入主节点(Primary)，副节点从主节点同步写入数据，以保持所有复制集内存储相同的数据，提供数据可用性

### 如何理解MongoDB中的GridFS机制，MongoDB为何使用GridFS来存储文件？

GirdFS是一种将大型文件存储在MongoDB中的文件规范。使用GridFS可以将大文件分隔成多个小文档存放，这样我们就能够有效的保存大文档，而且解决了BSON对象有限制的问题

### 索引类型有哪些？

单字段索引(Single Field Indexes)

复合索引(Compound Indexes)

多健索引(Multikey Indexes)

全文索引(text Indexes)

Hash 索引(Hash Indexes)

通配符索引(Wildcard Index)

2dsphere索引(2dsphere Indeses)

### 什么是master或primary？

副本集只能有一个主节点能够确认写入操作来接收所有写操作，并记录其操作日志中的数据集的所有更改(记录在oplog中)。在集群中，当主节点(master)失效，Secondary节点会变为master

### 复制集节点类型有哪些？

优先级0型(Priority 0)节点

隐藏型(Hidden)节点

延迟型(Delayed)节点

投票型(Vote)节点以及不可投票节点

### MongoDB的常见命令如下

1. db.help(); Help  查看命令提示
2. use yourDB;  切换/创建数据库
3. show dbs;  查询所有数据库
4. db.dropDatabase();  删除当前使用数据库
5. db.getName();  查看当前使用的数据库
6. db.version();  当前db版本
7. db.addUser("name");  添加用户
8. db.addUser("userName", "pwd123", true);
9. show users;  显示当前所有用户
10. db.removeUser("userName");  删除用户
11. db.yourColl.count();  查询当前集合的数据条数

### python中调用Mongo数据库的包叫什么？

Pymongo

### 分析器在MongoDB中的作用是什么？

MongoDB中包括了一个可以显示数据库中每个操作性能特点的数据库分析器。通过这个分析器你可以找到比预期慢的查询(或写操作);利用这一信息，比如，可以确定是否需要添加索引

### 怎么查看MongoDB正在使用的链接？

db._adminCommand("connPoolStats");

### 分片(sharding)和复制(replication)是怎样工作的？

分片可能是单一的服务器或者集群组成，推荐使用集群

### 数据在什么时候才会扩展到多个分片(shard)里?

mongodb分片是基于区域的，所以一个集合的所有对象都放置在同一个块中，只有当存在多余一个块的时候，才会有多个分片获取数据的选项

### MongoDB中的命名空间是什么意思

MongoDB内部有预分配空间的机制，每个预分配的文件都用0进行填充。

数据文件每新分配一次，它的大小都是上一个数据文件大小的2倍，每个数据文件最大2G。

MongoDB每个集合和每个索引都对应一个命名空间，这些命名空间的元数据集中在16M的*.ns文件中，平均每个命名占用约628字节，也即整个数据库的命名空间的上线约为24000。

如果每个集合有一个索引(比如默认的_id索引)，那么最多可以创建12000个集合。如果索引数更多，则可创建的集合数就更少了。同时，如果集合数太多，一些操作也会变慢。

要建立更多的集合的话，MongoDB也是支持的，只需要在启动时加上"-nssize"参数，这样对应数据库的命名空间文件就可以变得更大以便保存更多的命名。这个命名空间文件(.ns文件)最大可以为2G。

每个命名空间对应的盘区不一定是连续的。与数据文件增长相同，每个命名空间对应的盘区大小都是随分配次数不断增长的。目的是为了平衡命名空间浪费的空间与保持一个命名空间数据的连续性。

需要注意的一个命名空间freelist,这个命名空间用于记录不再使用的盘区(被删除的Collection或索引)。每当命名空间需要分配新盘区时，会先查看freelist，这个命名空间用于记录不再使用的盘区(被删除的Collection或索引)。每当命名空间需要分配新盘区时，会先查看freelist，这个命名空间用于记录不再使用的盘区(被删除的Collection或索引)。每当命名空间需要分配新盘区时，会先查看freelist是否有大小合适的盘区可以使用，如果有就回收空闲的磁盘空间。

### 在MongoDB中如何查看数据库列表

使用命令"show dbs"

### 什么是复杂

复杂是将数据同步到多个服务器的过程，通过多个数据副本存储到多个服务器上增加数据可用性。复制可以保障数据的安全性，灾难恢复，无需停机维护(如备份，重建索引，压缩)，分布式读取数据。

### 在MongoDB中如何在集合中插入一个文档

要想将数据插入MongoDB集合中，需要使用insert()或save()方法。

db.collectionName.insert({"key": "value"})

db.collectionName.save({"key": "value"})

### 在MongoDB中如何除去一个数据库Collection Methods24.在MongoDB中如何除去一个数据库

MongoDB的dropDatabase()命令用于删除已有数据库。

db.dropDatabase()

### 在MongoDB中如何创建一个集合

在MongoDB中，创建集合采用db.createCollection(name, options)方法。options是一个用来指定集合配置的文档。

db.createCollection("collectionName") db.createCollection()  -  MongoDB Manual > db.createCollection()

### 在MongoDB中如何查看一个已经创建的集合

可以使用show collections 查看当前数据库中的所有集合清单

### 你说的NoSQL数据库是什么意思？NoSQL与RDBMS直接有什么区别？为什么要使用和不使用NoSQL数据库？说一说NoSQL数据库的几个优点？

NoSQL是非关系型数据库，NoSQL = Not Only SQL

关系型数据库采用的结构化的数据，NoSQL采用的是键值对的方式存储数据。

在处理非结构化/半结构化的大数据时；在水平方向上进行扩展时；随时应对动态增加的数据项时可以优先考虑使用NoSQL数据库。

再考试数据库的成熟度；支持；分析和商业智能；管理及专业性等问题时，应优先考虑关系型数据库

### NoSQL数据库有哪些类型?

例如：MongoDB，Cassandra，CouchDB，Hypertable，Redis，Riak，HBASE，Memcache

### 名字空间(namespace)是什么？

MongoDB存储BSON对象在丛集(collection)中。数据库名字和丛集名字以句点连接起来叫做名字空间(namespace)。

### 如果用户移除对象的属性，该属性是否从存储层中删除？

是的，用户移除属性然后对象会重新保存(re-save())。

### 能否使用日志特征进行安全备份？

是的。

### 允许空值null吗？

对于对象成员而言，是的。然而用户不能够添加空值(null)到数据库丛集(collection)因为空值不是对象。然而用户能够添加空对象{}。

### 更新操作立刻fsync到磁盘？

不会，磁盘写操作默认是延迟执行的。写操作可能在两三秒(默认在60秒内)后到达磁盘。例如，如果一秒内数据库收到一千个对一个对象递增的操作，仅刷新磁盘一次。(注意，尽管fsync选项在命令行和经过getLastError_old是有效的)

### 如何执行事务/加锁？

MongoDB没有使用传统的锁或者复杂的带回滚的事务，因为它设计的宗旨是轻量，快速以及可预计的高性能。可以把它类比成MySQLMyISAM的自动提交模式。通过精简对事务的支持，性能得到了提升，特别是在一个可能会穿过多个服务器的系统里。

### 为什么我的数据文件如此庞大？

MongoDB会积极的预分配预留空间来防止文件系统碎片。

### 启用备份故障恢复需要多久？

从备份数据库声明主数据库宕机到选出一个备份数据库作为新的主数据库将花费10到30秒时间。这期间在主数据库上的操作将会失败-包括写入和强一致性读取(strong consistent read)操作。然而，你还能在第二数据库上执行最终一致性查询(eventually consistent query)(在slaveOK模式下)，即使在这段时间里。

### 什么是secondary或slave?

Seconday从当前的primary上复制相应的操作。它是通过跟踪复制oplog(local.oplog.rs)做到的。

### 我必须调用getLastError来确保写操作生效吗？

不用，不管你有没有调用getLastError(又叫"Safe Mode")服务器做的操作都一样。调用getLastError只是为了确认写操作成功提交了。当然，你经常想得到确认，但是写操作的安全性和是否生效不是由这个决定的。

### 我应该启动一个集群分片(sharded)还是一个非集群分片的MongoDB环境？

为开发便捷起见，我们建议以非集群分片(unsharded)方式开始一个MongoDB环境，除非一台服务器不足以存放你的初始数据集。从非集群分片升级到集群分片(sharding)是无缝的，所以在你的数据集还不是很大的时候没必要考虑集群分片(sharding)。

### MongoDB支持存储过程吗？如果支持的话，怎么用？

MongoDB支持存储过程，它是JavaScript写的，保留db.system.js表中。

### 如果一个分片(Shard)停止或很慢的时候，发起一个查询会怎样？

如果一个分片停止了，除非查询设置了"Partial"选项，否则查询会返回一个错误。如果一个分片响应很慢，MongoDB会等待它的响应。

### 如何理解MongoDB中的GridFS机制，MongoDB为何使用GridFS来存储文件？

GridFS是一种将大型文件存储在MongoDB中的文件规范。使用GridFS可以将大文件分隔成多个小文档存放，这样我们能够有效的保存大文档，而且解决了BSON对象有限制的问题。

### 什么是集合

集合就是一组MongoDB文档。它相当于关系型数据库(RDBMS)中的表这种概念。集合位于单独的一个数据库中。一个集合内的多个文档可以有多个不同的字段。一般来说，集合中的文档都有着相同或相关的目的。

### 什么是文档

文档由一组key value组成。文档是动态模式，这意味着同一集合里的文档不需要有相同的字段和结构。在关系型数据库中table中的每一条记录相当于MongoDB中的一个文档。

### 什么是"mongod"

mongod是处理MongoDB系统的主要进程。它处理数据请求，管理数据存储，和执行后台管理操作。当我们运行mongod命令意味着正在启动MongoDB进程，并且在后台运行。

### "mongod"参数有什么

传递数据库存储路径，默认是"/data/db"

端口号 默认是 "27017"

### MongoDB那个命令创建数据库

MongoDB用use+数据库名称的方式来创建数据库。

use会创建一个新的数据库，如果该数据库存在，则返回这个数据库。

> ### Redis中list底层实现有哪几种？有什么区别？

列表对象的编码可以是ziplist或者linked list

ziplist是一种压缩链接，它的好处是更能节省内存空间，因为它所存储的内容都是在连续的内存区域当中的。当列表对象元素不大，每个元素也不大的时候，就采用ziplist存储。但当数据量过大时就ziplist就不是那么好用了。因为为了保证它存储内容在内存中的连续性，插入的复杂度是O(N),即每次插入都会重新进行realloc。如下图所示，对象结构中ptr所指向的就是一个ziplist。整个ziplist只需要malloc一次，它们在内存中是一块连续的区域。

> ### 怎样解决数据库高并发的问题？

解决数据库高并发的常见方案：

1. 缓存式的web应用程序架构：

​  在Web层和DB(数据库)层之间加一层cache层，主要目的：减少数据库读取负担，提高数据读取速度。cache存取的媒介是内存，可以考虑采用分布式的cache层，这样更容易破除内存容量的限制，同时增加了灵活性。

2. 增加Redis缓存数据库：
3. 增加数据库索引
4. 页面静态化：

​  效率最高、消耗最小的就是纯静态化的html页面，所以我们尽可能使我们的网站上的页面采用静态页面来实现，这个最简单的方法其实也是最有效的方法。用户可以直接获取页面，不用像MVC结构走那么多流程，比较适用于页面信息大量被前台程序调用，但是更新频率很小的情况。

5. 适用存储过程：

​  处理一次请求需要多次访问数据库的操作，可以把操作整合到存储过程，这样只要一次数据库访问就可以了

> ### Redis数据库，内容是以何种结构存放在Redis中的？

String(字符串)  Hash(哈希)  List(列表)  Set(集合)  Zset(sortedset:有序集合)

> ### Redis的并发竞争问题怎么解决？

方案一：可以使用独占锁的方式，类似操作系统的mutex机制，不过实现相对复杂，成本较高

方案二：使用乐观锁的方式进行解决(成本较低，非阻塞，性能较高)

> ### 如何用乐观锁方式进行解决？

本质上是假设不会进行冲突，使用redis的命令watch进行构造条件

> ### MySQL和Redis高可用性体现在哪些方面？

- MySQL Replication 是MySQL官方提供的主从同步方案，用于将一个MySQL实例的数据，同步到另一个实例中。Replication为保证数据安全做了重要的保证，也是现在运用最广的MySQL容灾方案。Replication用两个或以上的实例搭建了MySQL主从复制集群，提供单点写入，多点读取的服务，实现了读的scaleout。
- Sentinel是Redis官方为集群提供的高可用解决方案。在实际项目中可以使用sentinel去做Redis自动故障转移，减少人工介入的工作量。另外sentinel也给客户端提供了监控消息的通知，这样客户端就可根据消息类型去判断服务器的状态，去做对应的适配操作。
- 下面是Sentinel主要功能列表：

​  Monitoring：Sentinel持续检查集群中的master、slave状态，判断是否存活。

​  Notification：在发现某个Redis实例死的情况下，Sentinel能通过API通知系统管理员或其他程序脚本

​  Automatic failover：如果一个master挂掉后，sentinel立马启动故障转移，把某个slave提升为master。其他的slave重新配置指向新的master。

​  Configuration provider：对于客户端来说，sentinel通知是有效可信赖的。客户端会连接sentinel去请求当前master的地址，一旦发生故障sentinel会提供新地址给客户端。

> ### Redis和MongoDB的优缺点

MongoDB和Redis都是NoSQL，采用结构型数据存储。二者在使用场景中，存在一定的区别，这也主要由于二者在内存映射的处理过程，持久化的处理方法不同。MongoDB建议集群部署，更多的考虑到集群方案，Redis更偏重于进程顺序写入，虽然支持集群，也仅限于-从模式。

Redis优点：

- 读写性能优异
- 支持数据持久化，支持AOF和RDB两种持久化方式
- 支持主从复制，主机会自动将数据同步到从机，可以进行读写分离
- 数据结构丰富：支持string、hash、set、sortedset、list等数据结构

缺点：

- Redis不具备自动容错和恢复功能，主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的IP才能恢复
- 主机宕机，宕机前有部分数据未能及时同步到从机，切换IP后还会引入数据不一致的问题，降低了系统的可用性
- Redis的主从复制采用全量复制，复制过程中主机会fork出一个子进程对内存做一份快照，并将子进程的内存快照保存为文件发送给从机，这一过程需要确保主机有足够多的空余内存。若快照文件较大，对集群的服务能力会产生较大的影响，而且复制过程是在从机新加入集群或者从机和主机网络断开重连时都会进行，也就是网络波动都会造成主机和从机间的一次全量的数据复制，这对实际的系统运营造成了不小的麻烦。
- Redis较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。为避免这一问题，运维人员在系统上线时必须确保有足够的空间，这对资源造成了很大的浪费。

MongoDB优点：

- 弱一致性（最终一致），更能保证用户的访问速度
- 文档结构的存储方式，能够更便捷的获取数据
- 内置GridFS，高效存储二进制大对象（比如照片和视频）
- 支持复制集、主备、互为主备、自动分片等特性
- 动态查询
- 全索引支持，扩展到内部对象和内嵌数组

缺点：

- 不支持事务
- MongoDB占用空间过大
- 维护工具不够成熟

> ### Redis基本类型、相关方法

Redis支持五种数据类型：string（字符串）、hash（哈希）、list（列表）、set（集合）、及zset(sortedset：有序集合)

一、 String

1. String是Redis最为常用的一种数据类型，String的数据结构为key/value类型，String可以包含任何数据。
2. 常用命令：set  get  decr  incr  mget 等

二、Hash

1. Hash类型可以看成是一个key/value都是String的Map容器
2. 常用命令：hget  hset hgetall 等

三、List

1. List用于存储一个有序的字符串列表，常用的操作是想队列两端添加元素或者获得列表的某一片段
2. 常用命令：lpush  rpush  lpop rpop lrange等

四、Set

1. Set可以理解为一组无序的字符集合，Set中相同的元素是不会重复出现的，相同的元素只保留一个。
2. 常用命令：sadd  spop  smenbers  sunion等

五、Sorted Set(有序集合)

1. 有序集合是在集合的基础上为每一个元素关联一个分数，Redis通过分数为集合中的成员进行排序
2. 常用命令：zadd  zrange  zrem  zcard等

> ### Redis的事务（不遵从mysql的ACID）

一、 Redis事务允许一组命令在单一步骤中执行。事务有两个属性，说明如下：

1. 事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断
2. Redis事务是原子的。原子意味着要么所有的命令都执行，要么都不执行

二、一个事务从开始到执行会经历以下三个阶段：

1. 开始事务
2. 命令入队
3. 执行事务

> ### Redis的使用场景有哪些？

1. 取最新N个数据的操作
2. 排行榜应用，取TOP N操作
3. 需要精准设定过期时间的应用
4. 计数器应用
5. uniq操作，获取某段时间所有数据排重值
6. Pub/Sub构建实时消息系统
7. 构建队列系统
8. 缓存

> ### Redis默认端口，默认过期时间，Value最多可以容纳的数据长度

1. 默认端口：6379
2. 默认过期时间：可以说永不过期，一般情况下，当配置中开启了超出最大内存限制就写磁盘的话，那么没有设置过期时间的key可能会被写到磁盘上。假如没设置，那么REDIS将使用LRU机制，将内存中的老数据删除，并写入新数据。
3. Value最多可以容纳的数据长度是：512M

> ### Redis有多少个库

Redis一个实例下有16个库

> ### Redis如何实现主从复制？以及数据同步机制

和Mysql主从机制的原因一样，Redis虽然读取写入的速度都特别快，但是也会产生读压力特别大的情况。为了分担读压力，Redis支持主从复制，Redis的主从结构可以采用一主多从或者级联结构，Redis主从复制可以根据是否是全量分为全量同步和增量同步。

Redis全量复制一般发生在Slave初始化阶段，这时Slave需要将Master上的所有数据都复制一份。具体步骤如下：

- 从服务器连接主服务器，发送SYNC命令；
- 主服务器接收到SYNC命令后，开始执行BGSAVE命令生成RDB文件并使用缓冲区记录此后执行的所有写命令；
- 主服务器BGSAVE执行完后，向所有从服务器发送快照文件，并在发送期间继续记录被执行的写命令；
- 从服务器收到快照文件后丢弃所有旧数据，载入收到的快照；
- 主服务器快照发送完毕后开始向从服务器发送缓冲区中的写命令；
- 从服务器完成对快照的载入，开始接收命令请求，并执行来自主服务器缓冲区的写命令；

增量同步

Redis增量复制是指Slave初始化后开始正常工作时主服务器发生的写操作同步到从服务器的过程。

增量复制的过程主要是主服务器每执行一个写命令就会向从服务器发送相同的写命令，从服务器接收并执行收到的写命令。

Redis主从同步策略

主从刚刚连接的时候，进行全量同步；全量同步结束后，进行增量同步。当然，如果有需要，slave在任何时候都可以发起全量同步。redis策略是：无论如何，首先会尝试进行增量同步，如不成功，要求从机进行全量同步。

主从复制的一些特点：

1. 采用异步复制；
2. 一个主redis可以含有多个从redis；
3. 每个从redis可以接收来自其他从redis服务器的连接；
4. 主从复制对于主redis服务器来说是非阻塞的，这意味着当从服务器再进行主从复制同步过程中，主redis仍然可以处理外界的访问请求；
5. 主从复制对于从redis服务器来说也是非阻塞的，这意味着，即使从redis在进行主从复制过程中也可以接受外界的查询请求，只不过这时候从redis返回的是以前的老数据，如果你不想这样，那么在启动redis时，可以在配置文件中进行设置，那么从redis在复制同步过程中来自外界的查询请求都会返回错误给客户端；(虽然说主从复制过程中对于从redis是非阻塞的，但是当从redis从主redis同步过来最新的数据后还需要将新数据加载到内存中，在加载到内存的过程中是阻塞的，在这段时间内的请求将会被阻塞，但是即使对于大数据集，加载到内存的时间也是比较多的)；
6. 主从复制提高了redis服务的扩展性，避免单个redis服务器的读写访问压力过大的问题，同时也可以给数据备份及冗余提供一种解决方案；
7. 为了编码主redis服务器写磁盘压力带来的开销，可以配置让主redis不再将数据持久化到磁盘，而是通过连接让一个配置从redis服务器及时将相关数据持久化到磁盘，不过这样会存在一个问题，就是主redis服务器一旦重启，因为主redis服务器数据为空，这时候通过主从同步可能导致从redis服务器上的数据也被清空；

redis中的sentinel的作用

帮助我们自动在主从直接进行切换

检测主从中主是否挂掉，且超过一半的sentinel检测到挂了之后才进行切换。

如果主修复好了，再次启动时候，会变成从。

启动主redis：

redis-server /etc/redis-6379.conf  启动主redis

redis-server /etc/redis-6380.conf  启动从redis

在Linux中：

找到/etc/redis-sentinel-8001.conf配置文件，在内部：

- 哨兵的端口  port = 8001
- 主redis的IP，哨兵个数的一半/1

找到/etc/redis-sentinel-8002.conf配置文件，在内部：

- 哨兵的端口  port = 8002
- 主redis的IP,1

启动两个哨兵

> ### 如何实现redis集群

redis集群、分片、分布式redis

redis-py-cluster

集群方案：

- redis cluster 官方提供的集群方案。
- codis, 豌豆荚技术团队
- tweproxy，Twiter技术团队

redis cluster 的原理?

- 基于分片来完成
- redis将所有能放置数据的地方创建了16384个哈希槽
- 如果设置集群的话，就可以为每个实例分片哈希槽：
- 192.168.1.20【0-5000】
- 192.168.1.21【5001-10000】
- 192.168.1.22【10001-16384】
- 以后想要在redis中写值时，set K1 123
- 将K1通过crc16的算法，将K1转换成一个数字。然后再将该数字和16384求余，如果得到的余数3000，那么就将该值写入到192.168.1.20实例中。

> ### redis中默认有多少哈希槽

16384

> ### 简述redis有哪几种持久化策略及比较

RDB：每隔一段时间对redis进行一次持久化。

- 缺点：数据不完整
- 优点：速度快

AOF：把所有命令保存起来，如果想得到重新生成的redis，那么就要把命令重新执行一次。

- 缺点：速度慢，文件比较大
- 优点：数据完整

> ### 列举redis支持的过期策略

voltile-lru：从已设置过期时间的数据集(server.db[i].expires)中挑选最近频率最少数据淘汰

volatile-ttl：从已设置过期时间的数据集(server.db[i].expires)中挑选将要过期的数据淘汰

volatile-random：从已设置过期时间的数据集(server.db[i].expires)中任意选择数据淘汰

allkeys-lru：从数据集(server.db[i].dict)中挑选最近最少使用的数据淘汰

allkeys-random：从数据集(server.db[i].dict)中任意选择数据淘汰

no-enviction(驱逐)：禁止驱逐数据

> ### 什么是codis及作用

Codis是一个分布式Redis解决方案，对于上层的应用来说，连接到Codis Proxy和连接原生的Redis Server没有明显的区别

(不支持的命令列表)，上层应用可以像使用单机的Redis一样使用，Codis底层会处理请求的转发，不停机的数据迁移等工作，所有后边的一切事情，对于前面的客户端来说是透明的，可以简单的认为后边连接的是一个内存无限大的Redis服务

> ### 写代码实现Redis事务操作

```python
import redis

pool = redis.ConnectionPool(host='10.211.55.4',port=6379)

conn = redis.Redis(connection_pool=pool)

# pipe = r.pipeline(transaction=False)

pipe = conn.pipeline(transaction=True)

# 开始事务
pipe.multi()
pipe.set('name','bendere')
pipe.set('role','sb')
# 提交
pipe.execute()
注意：咨询是否当前分布式redis是否支持事务

```

> ### Redis常用命令集

- 连接操作命令

```python
quit：关闭连接(connection)
auth：简单密码认证
help cmd：查看cmd帮助，例如：help quit
```

- 持久化

```bash
save：将数据同步保存到磁盘
bgsave：将数据异步保存到磁盘
lastsave：返回上次成功将数据保存到磁盘的Unix时戳
shundown：将数据同步保存到磁盘，然后关闭服务
```

- 远程服务控制

```python
info：提供服务器的信息和统计
monitor：实时转储收到的请求
slaveof：改变复制策略设置
config：在运行时配置Redis服务器
```

- 对value操作的命令

```bash
exists(key)：确认一个key是否存在
del(key)：删除一个key
type(key)：返回值的类型
keys(pattern)：返回满足给定pattern的所有key
randomkey：随机返回key空间的一个
keyrename(oldname,newname)：重命名key
dbsize：返回当前数据库中key的数目
expire：设定一个key的活动时间(s)
ttl：获得一个key的活动时间
select(index)：按索引查询
move(key,dbindex)：移动当前数据库中的key到dbindex数据库
flushdb：删除当前选择数据库中的所有key
flushall：删除所有数据库中的所有key
```

- String

```bash
set(key,value)：给数据库中名称为key的string赋予值value
get(key)：返回数据库中名称为key的string的value
getset(key,value)：给名称为key的string赋予上一次的value
mget(key1,key2,...,keyN)：返回库中多个string的value
setnx(key,value)：添加string，名称为key，值为value
setex(key,time,value)：向库中添加string，设定过期时间time
mset(keyN,valueN)：批量设置多个string的值
msetnx(keyN,valueN)：如果所有名称为keyi的string都不存在
incr(key)：名称为key的string增1操作
incrby(key,integer)：名称为key的string增加integer
decr(key)：名称为key的string减1操作
decrby(key,integer)：名称为key的string减少integer
append(key,value)：名称为key的string的值附件value
substr(key,start,end)：返回名称为key的string的value的字符串
```

- List

```bash
rpush(key,value)：在名称为key的list尾部添加一个值为value的元素
lpush(key,value)：在名称为key的list头部添加一个值为value的元素
llen(key)：返回名称为key的list的长度
lrange(key,start,end)：返回名称为key的list中start至end之间的元素
ltrim(key,start,end)：截取名称为key的list
lindex(key,index)：返回名称为key的list中index位置的元素
lset(key,index,value)：给名称为key的list中index位置的元素赋值
lrem(key,count,value)：删除count个key的list中值为value的元素
lpop(key)：返回并删除名称为key的list中的首元素
rpop(key)：返回并删除名称为key的list中的尾元素
blpop(key1,key2,...,keyN,timeout)：lpop命令的block版本
brpop(key1,key2,...,keyN,timeout)：rpop命令的block版本
rpoplpush(srckey,dstkey)：返回并删除名称为srckey的list的尾元素，并将该元素添加到名称为dstkey的list的头部
```

- Set

```bash
sadd(key,member)：向名称为key的set中添加元素member
srem(key,menber)：删除名称为key的set中元素member
spop(key)：随机返回并删除名称为key的set中一个元素
smove(srckey,dstkey,member)：移到集合元素
scard(key)：返回名称为key的set的基数
sismemeber(key,member)：member是否是名称为key的set的元素
sinter(key1,key2,...keyN)：求交集
sinterstore(dstkey,(keys))：求交集并将交集保存到dstkey的集合
sunion(key1,(keys))：求并集
sunionstore(dstkey,(keys))：求并集并将并集保存到dstkey的集合
sdiff(key1,(keys))：求差集
sdiffstore(dstkey,(keys))：求差集并将差集保存到dstkey的集合
smembers(key)：返回名称为key的set的所有元素
srandmember(key)：随机返回名称为key的set的一个元素
```

- Hash

```bash
hset(key,field,value)：向名称为key的hash中添加元素field
hget(key,field)：返回名称为key的hash中field对应的value
hmget(key,(fields))：返回名称为key的hash中fieldi对应的value
hmset(key,(fields))：向名称为key的hash中添加元素field
hincrby(key,field,integer)：将名称为key的hash中field的value增加integer
hexists(key,field)：名称为key的hash中是否存在健为field的域
hdel(key,field)：删除名称为key的hash中健为field的域
hlen(key)：返回名称为key的hash中元素个数
hkeys(key)：返回名称为key的hash中所有健
hvals(key)：返回名称为key的hash中所有健对应的value
hgetall(key)：返回名称为key的hash中所有的健(field)及其对应的value
```

> ### Redis高级应用

1. #### 安全性

​ 设置客户端连接后进行任何操作指定前需要密码，一个外部用户可以在一秒钟进行150W次访问，具体操作密码修改设置redis.conf里面的requirepass属性给予密码，当然我这里给的是primos

之后如果想操作可以采用登录的时候就授权使用：

sudo/opt/java/redis/bin/redis-cli -a primos

或者是进入以后auth primos 然后就可以随意操作了

2. #### 主从复制

做这个操作的时候我准备了两个虚拟机，ip分别是192.168.15.128和192.168.15.133

通过主从复制可以允许多个slave server拥有和master server相同的数据库，副本具体配置是在slave上面配置slave slaveof 192.168.15.128 6379  masterauth primos 如果没有主从同步那么就检查一下是不是防火墙的问题，我用的是ufw，设置一下sudo ufw allow 6379就可以了这个时候可以通过info查看具体的情况

3. #### 事务处理

​ redis对事务的支持还比较简单，redis只能保证一个client发起的事务中的命令可以连续执行，而中间不会插入其他client的命令。当一个client在一个连接中发出multi命令时，这个连接会进入一个事务的上下文，连接后续命令不会立即执行，而是先放到一个队列中，当执行exec命令时，redis会顺序的执行队列中的所有命令。

比如我下面的一个例子

set age 100

multi

set age 10

set age 20

exec

get age  -- 这个内容就应该是20

multi

set age 20

set age 10

exec

get age  -- 这个时候的内容就成了10，充分体现了一下按照队列顺序执行的方式discard 取消所有事物，也就是事物回滚，不过在redis事物执行有个别错误的时候，事物不会回滚，会把不错误的内容执行，错误的内容直接放弃，目前最新的是2.6.7也有这个问题的乐观锁watch key如果没watch的key有改动那么outdate的事务是不能执行的

4. #### 持久化机制

​ redis是一个支持持久化的内存数据库

snapshotting快照方式，默认的存储方式，默认写入dump.rdb的二进制文件中，可以配置redis在n秒内如果超过m个key被修改过就自动做快照append-only file aof方式，使用aof时候redis会将每一次的函数都追加到文件中，当redis重启时会重新执行文件中的保存的写命令在内存中。

5. 发布订阅消息  sbusribe publish操作，其实就类似Linux下面的消息发布
6. 虚拟内存的使用可以配置vm功能，保存路径，最大内存上线，页面多少，页面大小，最大工作线程临时修改ip地址ifconfig eth0 192.168.15.129

### redis-cli参数

```bash
Usage: redis-cli [OPTIONS] [cmd[arg[arg...]]]
 -h<hostname>  Serverhostname(default:127.0.0.1)
 -p<port>    Server port (default:6379)
 -s<socket>  Server socket (override hostname and port)
 -a<password>  Password to use when connecting to the server
 -r<repeat>  Execute specified command N times
 -i<interval>  When -r is used,waits<interval>seconds per command.
    It is possible to specify sub-second times like-i 0.1
 -n<db>  Database number
 -x    Read last argument from STDIN
 -D<delimiter>  Multi-bulk delimiter in for raw formatting(default: \n)
 -c  Enable cluster mode(follow-ASK and -MOVED redirections)
 --raw  Use raw formatting for replies (default when STDOUT is not atty)
 --latency  Enter a special mode continuously sampling latency
 --slave  Simnlate a slave showing commands received from the master
 --pipe  Transfer raw Redis protocol from stdin to server
 --bigkeys  Sample Redis keys looking for big keys
 --eval<file>  Send an EVAL command using the Lua script at<file>
 --help  Output this help and exit
 --version  Output version and exit
 
Examples:
 cat/etc/passwd | redis-cli -x set mypasswd
 redis-cli get mypasswd
 redis-cli -r 100 lpush mylist x
 redis-cli -r 100 -i 1 info | grep used_memory_human:
 redis-cli --eval myscript.lua key1 key2, arg1 arg2 arg3
 (Note:when using --eval the comma separates KEYS[] from ARGV[] items)
  
```

常用命令：

1）查看keys个数

keys *   // 查看所有keys

keys prefix_*    // 查看前缀为"prefix_"的所有keys

2）清空数据库

flushdb    // 清除当前数据库的所有keys

flushall  // 清除所有数据库的所有keys

> ### 什么是Python

Python是一种编程语言，它有对象、模块、线程、异常处理和自动内存管理，可以加入其它语言的对比。

Python是一种解释型语言，Python在代码运行之前不需要解释。

Python是动态类型语言，在声明变量时，不需要说明变量的类型。

Python适合面向对象的编程，因为它支持通过组合与继承的方式定义类。

在Python语言中，函数是第一类对象。

Python代码编写快，但是运行速度比编译型语言通常要慢。

Python用途广泛，常被用作"胶水语言"，可帮助其他语言和组件改善运行状况。

使用Python，程序员可以专注于算法和数据结构的设计，而不用处理底层的细节。

> ### 谈谈对Python和其他语言的区别

Python属于解释型语言，当程序运行时，是一行一行的解释，并运行，所以调试代码很方便，开发效率高，还有龟叔给Python定位是任其自由发展、优雅、明确、简单，所以在每个领域都有建树，所以它有着非常强大的第三方库。

特点：

语法简洁优美，功能强大，标准库与第三方库都非常强大，而且应用领域也非常广，可移植性，可扩展性，可嵌入性

缺点：

运行速度慢，

- 解释型

​ python php

- 编译型

​ c java c#

Python弱类型

与php相比：python标准包直接提供了工具，并且相对于php代码更易于维护；

Python的优势：

1. Python易于学习
2. 用少量的代码构建出很多功能（高效的高级数据结构）
3. Python拥有最成熟的程序包资源库之一
4. Python完全支持面向对象
5. Python是跨平台且开源的
6. 动态类型

> ### Python的解释器种类以及相关特点

#### CPython

是官方版本的解释器：CPython。是使用C语言开发的，所以叫CPython。在命令行下运行python就是启动CPython解释器。CPython是使用最广的Python解释器。教程的所有代码也都在CPython下执行。

#### IPython

IPython是基于CPython之上的一个交互式解释器，也就是说，IPython只是在交互方式上有所增强，但是执行Python代码的功能和CPython是完全一样的。CPython用>>>作为提示符，而IPython用In[序号]:作为提示符。

#### PyPy

由Python写的解释器，它的执行速度是最快。PyPy采用JIT技术，对Python代码进行动态编译（注意不是解释），绝大部分Python代码都可以在PyPy下运行，但是PyPy和CPython有一些是不同的，这就导致相同的Python代码在两种解释器下执行可能会有不同的结果。

#### Jython

Jython是运行在Java平台上的Python解释器，可以直接把Python代码编译成Java字节码执行。

#### IronPython

IronPython和Jython类型，只不过Ironpython是运行在.Net平台上的Python解释器，可以直接把Python代码编译成.Net的字节码。

#### 小结

Python的解释器很多，但使用最广泛的还是CPython。如果要和Java或.Net平台交互，最好的办法不是用Jython或IronPython，而是通过网络调用来交互，确保各程序之间的独立性。

> ### Python递归的最大层数

998 主要还是要和执行的机器有关

> ### ascii unicode utf-8 gbk 区别

python2内容进行编码（默认ascii），而python3对内容进行编码的默认为utf-8。

ASCII 最多只能用8位来表示（一个字节），即：2**8=256，所以，ASCII码最多只能表示256个符号。

Unicode 万国码，任何一个字符==两个字节

utf-8  万国码的升级版 一个中文字符==三个字节 英文是一个字节 欧洲的是2个字节

gbk  国内版本 一个中文字符==2个字节 英文是一个字节

gbk 转utf-8 需通过媒介 Unicode

> ### Python2和Python3的区别

核心类差异，废弃类差异，修改类差异，第三方工具包差异，工具安装问题，

- #### 核心类差异

1. Python3对Unicode字符的原生支持。

​ Python2中使用ASCII码作为默认编码方式导致string有两种类型str和Unicode，Python3只支持Unicode的string。Python2和Python3字节和字符对应关系为：

2. Python3采用的是绝对路径的方式进行import。

​ Python2中相对路径的import会导致标准库导入变得困难（想像一下，同一目录下有file.py，如何同时导入这个文件和标准库file）。Python3中这一点将被修改，如果还需要导入同一目录的文件必须使用绝对路径，否则只能使用相对导入的方式来进行导入。

3. Python2中存在老式类和新式类的区别，Python3统一采用新式类。新式类声明要求继承object，必须用新式类应用多重继承。
4. Python3使用更加严格的缩进。Python2的缩进机制中，1个tab和8个space是等价的，所以在缩进中可以同时允许tab和space在代码中共存。这种等价机制会导致部分IDE使用存在问题。

​ Python3中1个tab只能找另外一个tab替代，因此tab和space共存会导致报错：TabError:inconsistent use of tabs and spaces in indentation

- #### 废弃类差异

1. print语句被Python3废弃，统一使用print函数

2. exec语句被Python3废弃，统一使用exec函数

3. execfile语句被Python3废弃，推荐使用exec(open("./filename").read())

4. 不相等操作符"<>"被Python3废弃，统一使用"!="

5. long整数类型被Python3废弃，统一使用int

6. xrange函数被Python3废弃，统一使用range，Python3中range的机制也进行修改并提高了大数据集生成效率

7. Python3中这些方法不再返回list对象：dictionary关联的keys()、values()、items()、zip()、map()、filter()，但是可以通过list强行转换：

    1. mydict={"a":1,"b":2,"c":3}

    2. mydict.keys()  #<built-in method keys of dict object at 0x000000000040B4C8>

    3. list(mydict.keys())  # ['a','c','b']

8. 迭代器iterator的next()函数被Python3废弃，统一使用next(iterator)

9. raw_input函数被Python3废弃，统一使用input函数

10. 字典变量的has_key函数被Python废弃，统一使用in关键词

11. file函数被Python3废弃，统一使用open来处理文件，可以通过io.IOBase检查文件类型

12. apply函数被Python3废弃

13. 异常StandardError被Python3废弃，统一使用Exception

- #### 修改类差异

1. ##### 浮点数除法操作符"/"和"//"的区别

​  "/":

​ Python2：若为两个整型数进行运算，结果为整型，但若两个数中有一个为浮点数，则结果为浮点数；

​ Python3：为真除法，运算结果不再根据参加运算的数的类型

​  "//":

​ Python2：返回小于除法运算结果的最大整数；从类型上讲，与"/"运算符返回类型逻辑一致

​ Python3：和Python2运算结果一样

2. ##### 异常抛出和捕捉机制区别

​ Python2

 1. raise IOError, "file error"  # 抛出异常
 1. except NameError, err:  #捕捉异常

​ Python3

 1. raise IOError("file error")  # 抛出异常

 2. except NameError as err:  # 捕捉异常

 1. ##### for 循环中变量值区别

​  Python2，for 循环会修改外部相同名称变量的值

  1. i = 1
  1. print (\`comprehension:`,[i for i in range(5)])
  1. print (\`after: i =`, i)  # i = 4

​  Python3，for循环不会修改外部相同名称变量的值

  1. i = 1
  2. print (\`comprehension:`, [i for i in range(5)])
  3. print (\`after:i = `,i)  # i=1

4. ##### round函数返回值区别

​ Python2，round函数返回float类型值

  1. isinstance(round(15,5),int)  # True

​ Python3, round函数返回int类型值

 1. isinstance(round(15,5), float)  # True

5. ##### 比较操作符区别

​ Python2中任意两个对象都可以比较

 1. 11 < 'test' # True

​ Python3中只有同一数据类型的对象可以比较

 1. 11 <'test' # TypeError: unorderable types: int() < str()

- #### 第三方工具包差异

我们在pip官方下载源pypi搜索Python2.7和Python3.5的第三方工具包数可以发现，Python2.7版本对应的第三方工具类目数量是28523，Python3.5版本的数量是12475，这两个版本在第三方工具包支持数量差距相当大。我们从数据分析的应用角度列举了常见实用的第三方工具包(如下表)，并分析这些工具包在Python2.7和Python3.5的支持情况：

|   分类   |      工具名      |                用途                |
| :------: | :--------------: | :--------------------------------: |
| 数据收集 |      scrapy      |           网页采集，爬虫           |
| 数据收集 |   scrapy-redis   |             分布式爬虫             |
| 数据收集 |   selenium web   |          测试，仿真浏览器          |
| 数据处理 |  beautifulsoup   |     网页解释库，提供lxml的支持     |
| 数据处理 |     lxml xml     |               解释库               |
| 数据处理 |    xlrd excel    |              文件读取              |
| 数据处理 |    xlwt excel    |              文件写入              |
| 数据处理 |  xlutils excel   |          文件简单格式修改          |
| 数据处理 |  pywin32 excel   |    文件的读取写入及复杂格式定制    |
| 数据处理 | Python-docx Word |           文件的读取写入           |
| 数据分析 |      numpy       |        基于矩阵的数学计算库        |
| 数据分析 |      pandas      |        基于表格的统计分析库        |
| 数据分析 |      scipy       | 科学计算库，支持高阶抽象和复制模型 |
| 数据分析 |   statsmodels    |     统计建模和计量经济学工具包     |
| 数据分析 |   scikit-learn   |           机器学习工具库           |
| 数据分析 |      gensim      |         自然语言处理工具库         |
| 数据分析 |      jieba       |           中文分词工具库           |
| 数据存储 |   MySQL-python   |         mysql的读写接口库          |
| 数据存储 |   mysqlclient    |         mysql的读写接口库          |
| 数据存储 |    SQLAlchemy    |          数据库的ORM封装           |
| 数据存储 |     pymysql      |      sql server 的读写接口库       |
| 数据存储 |      redis       |          redis的读写接口           |
| 数据存储 |     PyMongo      |         MongoDB的读写接口          |
| 数据呈现 |    matplotlib    |         流行的数据可视化库         |
| 数据呈现 |     seaborn      |  美观的数据可视库，基于matplotlib  |
| 工具辅助 |     jupyter      | 基于web的python IDE 常用于数据分析 |
| 工具辅助 |     chardet      |            字符检查工具            |
| 工具辅助 |   ConfigParser   |          配置文件读写支持          |
| 工具辅助 |     requests     |        HTTP库，用于网络访问        |

- #### 工具安装问题

​ windows 环境

​ python2 无法按照mysqlclient。Python3无法按照MySQL-python、flup、functools32、Gooey、Pywin32、webencodings。

​ matplotlib在python3环境中安装保错：The following required packages can not bebuilt:freetype,png。需要手动下载安装源码包安装解决。

scipy在Python3环境中安装保错，numpy.distutils.system_info.NotFoundError,需要自己手工下载对应的安装包，依赖numpy，pandas必须严格根据python版本、操作系统、64位与否。

运行matplotlib后发现基础包numpy+mkl安装失败，需要自己下载，国内暂无下载源。

centos环境下Python2无法安装mysql-python和mysqlclient包，报错：EnvironmentError:mysql_config notfound,解决方案是安装mysql-devel包解决。

使用matplotlib报错：no module named_tkinter,安装Tkinter、tk-devel、tc-devel解决。

pywin32也无法在centos环境下安装。

> ### 关于Python程序的运行方面，有什么手段能提升性能

1. 使用多进程，充分利用机器的多核性能
2. 对于性能影响较大的部分代码，可以使用C和C++编写
3. 对于IO阻塞造成的性能影响，可以使用IO多路复用来解决
4. 尽量使用Python的内建函数
5. 尽量使用局部变量

> ### Python中的作用域

Python中，一个变量的作用域总是由在代码中被赋值的地方所决定。当Python遇到一个变量的话它会安装这样的顺序进行搜索：

本地作用域(Local) --->当前作用域被嵌入的本地作用域(Enclosing locals) --->全局/模块作用域(Global) -->内置作用域(Built-in)

> ### 什么是Python自省

Python自省是Python具有的一种能力，是程序员面向对象的语言所写的程序在运行时，能够获得对象的类Python型。Python是一种解释型语言，为程序员提供了极大的灵活性和控制力。

> ### 什么是Python的命名空间？

在Python中，所有的名字都存在于一个空间中，它们在该空间中存在和被操作--这就是命名空间。它就好像一个盒子，每一个变量名字都对应装着一个对象。当查询变量的时候，会从该盒子里面寻找相应的对象。

> ### print调用Python中底层的什么方法？

print方法默认调用sys.stdout.write方法，即往控制台打印字符串。

> ### 简述你对input()函数的理解?

在Python3中，input()获取用户输入，不论用户输入的什么，获取到的都是字符串类型的。

在Python2中有raw_input()和input()，raw_input()和Python3中的input()作用是一样的，input()输入的是什么数据类型的，获取到的就是什么数据类型的

> ### Range和xrange的区别？

两者用法相同，不同的是range返回的结果是一个列表，而xrange的结果是一个生成器，前者是直接开辟一块内存空间来保存列表，后者是边循环边使用，只有使用时才会开辟内存空间，所以当列表很长时，使用xrange性能要比range好。

> ### read、readline和readlines的区别？

read：读取整个文件

readLine：读取下一行，使用生成器方法

readlines：读取整个文件到一个迭代器以供我们遍历。

> ### 在except中return后还会不会执行finally中的代码？怎么抛出自定义异常？

会继续处理finally中的代码：用raise方法可以抛出自定义异常

> ### 介绍一下except的作用和用法？

except：# 捕获所有异常

except：<异常名>：# 捕获指定异常

except: <异常名 1，异常名 2>：捕获异常 1 或者异常2

except:<异常名>,<数据>：捕获指定异常及其附件的数据

except:<异常名 1,异常名 2>:<数据>：捕获异常名 1 或者异常名 2，及附件的数据

> ### 常用的Python标准库都有哪些？

os操作系统，time时间，random随机，pymysql连接数据库，threading线程，multiprocess进程，queue队列。

第三方库：

Django和flask也是第三方库，requests，virtualenv,selenium,scrapy,xadmin,celery,re,hashlib,md5.

常用的科学计算库（如Numpy, Scipy,Pandas）。

> ### Python里面如何生成所及数？

在Python中用于生成随机数的模块是random，在使用前需要import

> ### 说明一下os.path和sys.path分别代表什么？

os.path主要是用于对系统路径文件的操作

sys.path主要是对Python解释器的系统环境参数的操作（动态的改变Python解释器搜索路径）。

> ### Python中的os模块常见方法？

```python
os.remove()删除文件
os.rename() 重命名文件
os.walk()  生成目录树下的所有文件名
os.chdir()  改变目录
os.mkdir/makedirs  创建目录/多层目录
os.rmdir/removedirs  删除目录/多层目录
os.listdir()  列出指定目录的文件
os.getcwd()  取得当前工作目录
os.chmod()  改变目录权限
os.path.basename()  去掉目录路径，返回文件名
os.path.dirname()   去掉文件名，返回目录路径
os.path.join()  将分离的各部分组合成一个路径名
os.path.split()  返回(dirname(),basename())元组
os.path.splitext()  返回(filename,extension)元组
os.path.getatime\ctime\mtime  分别返回最近访问、创建、修改时间
os.path.getsize()   返回文件大小
os.path.exists()    是否存在
os.path.isabs()     是否为绝对路径
os.path.isdir()     是否为目录
os.path.isfile()    是否为文件
```

> ### Python的sys模块常用方法？

```python
sys.argv命令行参数List，第一个元素是程序本身路径
sys.modules.keys()  返回所有已经导入的模块列表
sys.exc_info()  获取当前正在处理的异常类型，exc_type、exc_traceback当前处理的异常详细信息
sys.exit(n)  退出程序，正常退出时exit(0) 
sys.hexversion  获取Python解释程序的版本值，16进制格式如：0x020403F0
sys.version  获取Python解释程序的版本信息
sys.maxint  最大的Int值
sys.maxunicode  最大的Unicode值
sys.modules  返回系统导入的模块字段，key是模块名，value是模块
sys.path  返回模块的搜索路径，初始化时使用PYTHONPATH环境变量的值
sys.platform 返回操作系统平台名称
sys.stdout  标准输出
sys.stdin  标准输入
sys.stderr  错误输出
sys.exc_clear()  用来清除当前线程所出现的当前的或最近的错误信息
sys.exec_prefix  返回平台独立的python文件安装的位置
sys.byteorder  本地字节规则的指示器，big-endian平台的值是'big',little-endian平台的值是'little'
sys.copyright  记录python版权相关的东西
sys.api_version  解释器的C的API版本
sys.version_info  元组则提供一个更简单的方法来使你的程序具备Python版本要求功能模块和包是什么？
在Python中，模块是搭建程序的一种方式。每一个Python代码文件都是一个模块，并可以引用其他的模块，比如对象和属性。
一个包含许多Python代码的文件夹是一个包。一个包可以包含模块和子文件夹。
```

> ### Python是强语言类型还是弱语言类型？

Python是强类型的动态脚本语言。

强类型：不允许不同类型相加

动态：不使用显示数据类型声明，且确定一个变量的类型是在第一次给它赋值的时候。

脚本语言：一般也是解释型语言，运行代码只需要一个解释器，不需要编译。

谈一下什么是解释性语言，什么是编译型语言？

计算机不能直接理解高级语言，只能直接理解机器语言，所以必须要把高级语言翻译成机器语言，计算机才能执行高级语言编写的程序。

解释性语言在运行程序的时候才会进行翻译。

编译型语言写的程序在执行之前，需要一个专门的编译过程，把程序编译成机器语言（可执行文件）。

> ### Python中有日志吗？怎么使用？

有！Python自带logging模块，调用logging。basicConfig()方法，配置需要的日志等级和相应的参数，Python解释器会按照配置的参数生成相应的日志。

> ### Python是如何进行类型转换的？

内建函数封装了各种转换函数，可以使用目标类型关键字强制类型转换，进制之间的转换可以用int('str',base='n')将特定进制的字符串转换为十进制，再用相应的进制转换函数将十进制转换为目标进制。

> ### Python数据类型

- 字典

​ dict：字典，字典是一组健(key)和值(value)的组合，通过健(key)进行查找，没有顺序，使用大括号"{}";例如：d={'a':24,'g':52,'i':12,'k':33}

- 字符串

​ str：字符串是Python中最常用的数据类型。我们可以使用引号(''或"")来创建字符串。

将字符串"k:1|k1:2|k2:3|k3:4",处理成Python字典：{k:1,k1:2,...}

1. str1 = "k:1|k1:2|k2:3|k3:4"
2. def str2dict(str1):
3. dict1={}
4. for iterms in str1.split('|')
5. key,value = items.split(':')
6. dict1[key] = value
7. return dict1

- 列表

```python
 list：是Python中使用最频繁的数据类型，在其他语言中通常叫做数组，通过索引进行查找，使用方括号"[]",列表是有序的集合。应用场景：定义列表使用[]定义，数据之间使用","分割。

列表的索引从0开始：索引就是数据在列表中的位置编号，索引又可以被称为下标。

【注意】：从列表中取值时，如果超出索引范围，程序会产生异常。

例子：name_list = ["zhangsan","lisi","wangwu","zhaoliu"]

增加列表元素：
列表名.insert(index,数据)：在指定位置插入数据（位置前有空元素会补位）。

列表名.append(数据)：在列表的末尾追加数据（最常用的方法）。

列表.extend(Iterable)：将可迭代对象中的元素追加到列表。

取值和修改
取值：列表名[index]：根据下标来取值。
修改：列表名[index]：数据：修改指定索引的数据。
删除：del列表名[index]：删除指定索引的数据 del name_list[1]
列表名.remove(数据)：删除第一个出现的指定数据
name_list = ["zhangsan","lisi","wangwu","zhaoliu","lisi"]  # 删除第一次出现lisi的数据
name_list.remove("lisi")
列表名.pop()：删除末尾的数据，返回值：返回被删除的元素。
name_list = ["zhangsan","lisi","wangwu","zhaoliu"]  # 删除最后一个元素zhaoliu并将元素zhaoliu返回name_list.pop()
列表名.pop(index)：删除指定索引的数据，返回被删除的元素。
name_list = ["zhangsan","lisi","wangwu","zhaoliu"]  # 删除索引为1的数据lisi
name_list.pop(1)
列表名.clear()：清空整个列表的元素
列表名.sort()：升序排序 从小到大
列表名.sort(reverse=True)：降序排序 从大到小
列表名.reverse()：列表逆序、反转
len(列表名)：得到列表的长度
列表名.count(数据)：数据在列表中出现的次数
列表名.index(数据)：数据在列表中首次出现时的索引，没有查找会报错
if 数据 in 列表：判断列表中是否包含某元素
a = [11,22,33,44,55]
if 33 in a:
    print("找到了...")
使用while循环：
1. a = [11,22,33,44,55]
2. i = 0
3. while i < len(a):
4. print(a[i])
5. i += 1
使用for循环：
1. a = [11, 22, 33, 44, 55]
2. for i in a:
3. print(i)
```

- 元组

```python
tuple：元组，元组将多样的对象集合到一起，不能修改，通过索引进行查找，使用括号"()"
应用场景：把一些数据当做一个整体去使用，不能修改
```

- 集合

```python
set：set集合，在Python中书写方式的{},集合与之前列表、元组类似，可以存储多个数据，但是这些数据是不重复的。集合对象还支持union(联合)，intersection(交)，dirrerence(差)和sysmmetric_dirrerence(对称差集)等数学运算。
快速去除列表中的重复元素
1. In[4]: a = [11,22,33,33,44,22,55]
2. In[5]: set(a)
3. Out[5]:{11,22,33,44,55}
交集：共有的部分
1.In[7]: a = {11,22,33,44,55}
2.In[8]: b = {22,44,55,66,77}
3.In[9]: a&b
4.Out[9]:{22,44,55}
并集：总共的部分
1.In[11]: a = {11, 22, 33, 44, 55}
2.In[12]: b = {22,44,55,66,77}
3.In[13]: a | b
4.Out[13]:{11,22,33,44,55,66,77}
差集：另一个集合中没有的部分
1.In[15]: a = {11,22,33,44,55}
2.In[16]: b = {22,44,55,66,77}
3.In[17]: b-a
4.Out[17]:{66,77}
对称差集（在a或b中，但不会同时出现在二者中）
1. In[19]: a = {11,22,33,44,55}
2. In[20]: b = {22,44,55,66,77}
3. In[21]: a^b
4. Out[21]: {11,33,66,77}
```

> ### 说一下字典和json的区别？

字典是一种数据结构，json是一种数据的表现形式，字典的key值只要是能hash的就行，json的必须是字符串。

> ### 什么是可变、不可变类型？

可变不可变指的是内存中的值是否可以被改变，不可变类型指的是对象所在内存块里面的值不可以改变，有数值、字符串、元组；可变类型则是可以改变，主要有列表、字典。

> ### 存入字典里的数据有没有先后排序？

存入的数据不会自动排序，可以使用sort函数对字典进行排序。

> ### Python中类方法、类实例方法、静态方法有何区别？

类方法：是类对象的方法，在定义时需要在上方使用"@classmethod"进行装饰，形参为cls，表示类对象，类对象和实例对象都可调用；

类实例方法：是类实例化对象的方法，只有实例对象可以调用，形参为self，指代对象本身；

静态方法：是一个任意函数，在其上方使用"@staticmethod"进行装饰，可以用对象直接调用，静态方法实际上跟该类没有太大关系。

> ### Python的内存管理机制及调优手段？

内存管理机制：引用计数、垃圾回收、内存池。

#### 引用计数

引用计数是一种非常高效的内存管理手段，当一个Python对象被引用时其引用次数增加1，当其不再被一个变量引用时则计数减1，当引用计数等于0时对象被删除。

#### 垃圾回收

1. 引用计数

​ 引用计数也是一种垃圾收集机制，而且也是一种最直观，最简单的垃圾收集技术。当Python的某个对象的引用计数降为0时，说明没有任何引用指向该对象，该对象就成为要被回收的垃圾了。比如某个新建对象，它被分配给某个引用，对象的引用计数变为1.如果引用被删除，对象的引用计数为0，那么该对象就可以被垃圾回收。不过如果出现循环引用的话，引用计数机制就不再起有效的作用了

2. 标记清除

​ 如果两个对象的引用计数都为1，但是仅仅存在他们之间的循环引用，那么这两个对象都是需要被回收的，也就是说，它们的引用计数虽然表现为非0，但实际上有效的引用计数为0。所以先将循环引用摘掉，就会得出这两个对象的有效计数。

3. 分代回收

​ 从前面"标记-清除"这样的垃圾收集机制来看，这种垃圾收集机制所带来的额外操作实际上与系统中总的内存块的数量是相关的，当需要回收的内存块越多时，垃圾检测带来的额外操作就越多，而垃圾回收带来的额外操作就越少；反之，当需回收的内存块越少时，垃圾检测就将比垃圾回收带来更少的额外操作。

#### 内存池

1. Python的内存机制呈现金字塔形状，-1，-2层主要有操作系统进行操作；
2. 第0层是C中的malloc,free等内存分配和释放函数进行操作；
3. 第1层和第2层是内存池，有Python的接口函数PyMem_Malloc函数实现，当对象小于256K时有该层直接分配内存；
4. 第3层是最上层，也就是我们对Python对象的直接操作；

​ Python在运行期间会大量地执行malloc和free的操作，频繁地在用户态和核心态之间进行切换，这将严重影响Python的执行效率。为了加速Python的执行效率，Python引入了一个内存池机制，用于管理对小块内存的申请和释放。Python内部默认的小块内存与大块内存的分界点定在256个字节，当申请的内存小于256字节时，PyObject_Malloc会在内存池中申请内存；当申请的内存大于256字节时，PyObject_Malloc的行为将蜕化为malloc的行为。当然，通过修改Python源代码，我们可以改变这个默认值，从而改变Python的默认内存管理行为。

#### 调优手段(了解)

1. 手动垃圾回收
2. 调高垃圾回收阈值
3. 避免存活引用（手动解循环引用和使用弱引用）

> ### 内存泄露是什么？如何避免？

指由于疏忽或错误造成程序未能释放已经不再使用的内存的情况。内存泄露并非指内存在物理上的消失，而是应用程序分配某段内存后，由于设计错误，失去了对该段内存的控制，因而造成了内存的浪费。导致程序运行速度减慢甚至系统崩溃等严重后果。

有\__del__()函数的对象间的循环引用时导致内存泄露的主凶。

不使用一个对象时使用：del object 来删除一个对象的引用计数就可以有效防止内存泄露问题。

通过Python扩展模块gc来查看不能回收的对象的详细信息。

可以通过sys.getrefcount(obj)来获取对象的引用计数，并根据返回值是否为0来判断是否内存泄露。

> ### Python函数调用的时候参数的传递方式是值传递还是引用传递？

Python的参数传递有：位置参数、默认参数、可变参数、关键字参数。

函数的传值到底是值传递还是引用传递，要分情况：

不可变参数用值传递：

像整数和字符串这样的不可变对象，是通过拷贝进行传递的，因为你无论如何都不可能在原处改变不可变对象

可变参数是引用传递的：

比如像列表，字典这样的对象是通过引用传递、和C语言里面的用指针传递数组很相似，可变对象能在函数内部改变。

> ### 对缺省参数的理解？

缺省参数指在调用函数的时候没有传入参数的情况下，调用默认的参数，在调用函数的同时赋值时，所传入的参数会替代默认参数。

*args是不定长参数，他可以表示输入参数是不确定的，可以是任意多个。

**kwargs是关键字参数，赋值的时候是以健=值的方式，参数是可以任意多对，在定义函数的时候不确定会有多少参数会传入时，就可以使用两个参数。

> ### 为什么函数名字可以当做参数用？

Python中一切皆对象，函数名是函数在内存中的空间，也是一个对象

> ### Python中pass语句的作用是什么？

在编写代码时只写框架思路，具体实现还未编写就可以用pass进行占位，使程序不报错，不会进行任何操作。

> ### map函数和reduce函数？

1. 从参数方面来讲：

​ map()包含两个参数，第一个参数是一个函数，第二个是序列（列表或元组）。其中，函数（即map的第一个参数位置的函数）可以接收一个或多个参数。reduce()第一个参数是函数，第二个是序列（列表或元组）。但是，其函数必须接收两个参数。

2. 从对传进去的数值作用来讲：

​ map()是将传入的函数依次作用到序列的每个元素，每个元素都是独自被函数"作用"一次。

reduce()是将传入的函数作用在序列的第一个元素得到结果后，把这个结果继续与下一个元素作用（累积计算）。

> ### 递归函数停止的条件？

递归的终止条件一般定义在递归函数内部，在递归调用前要做一个条件判断，根据判断的结果选择是继续调用自身，还是return；返回终止递归。

终止的条件：

1. 判断递归的次数是否达到某一限定值
2. 判断运算的结果是否达到某个范围等，根据设计的目的来选择

> ### 回调函数，如何通信的？

回调函数是把函数的指针（地址）作为参数传递给另一个函数，将整个函数当做一个对象，赋值给调用的函数。

> ### Python主要的内置数据类型都有哪些？

内建类型：布尔类型、数字、字符串、列表、元组、字典、集合；

> ### 什么是lambda函数？有什么好处？

lambda函数是一个可以接收任意多个参数（包括可选参数）并且返回单个表达式值的函数

1. lambda函数比较轻便，即用即扔，很适合需要完成一项功能，但是此项功能只在此一处使用，连名字都很随意的情况下；
2. 匿名函数，一般用来给filter，map这样的函数式编程服务；
3. 作为回调函数，传递给某些应用，比如消息处理

​ 使用lambda函数能创建小型匿名函数。这种函数得名于省略了用def

> ### Python设计模式

单例，工厂，装饰器，生成器

#### 单例

```python
1 class A(object):
2 __instance = None
3 def __new__(cls,*args,**kwargs):
4 if cls.__instance is None:
5 cls.__instance = object.__new__(cls)
6 return cls.__instance
7 else:
8 return cls.__instance
```

#### 单例模式的应用场景有哪些?

单例模式应用的场景一般发现在以下条件下：

1. 资源共享的情况下，避免由于资源操作时导致的性能或损耗等。如日志文件，应用配置。
2. 控制资源的情况下，方便资源之间的互相通信。如线程池等。1.网站的计数器 2.应用配置 3.多线程池 4. 数据库配置，数据库连接池 5. 应用程序的日志应用...

#### 装饰器

对装饰器的理解，并写出一个计时器记录方法执行性能的装饰器？

```python
装饰器本质上是一个Python函数，它可以让其他函数在不需要做任何代码变动的前提下增加额外功能，装饰器的返回值也是一个函数对象。
1. import time 
2. def timeit(func):
3. def wrapper():
4. start = time.clock()
5. func()end = time.clock()
6. print 'used:', end=start
7. return wrapper
8. @timeit
9. def foo():
10. print'in foo()'foo()
```

> ### 解释一下什么是闭包？

在函数内部再定义一个函数，并且这个函数用到了外边函数的变量，那么将这个函数以及用到的一些变量称之为闭包。

> ### 函数装饰器有什么作用？

装饰器本质上是一个Python函数，它可以在让其他函数在不需要做任何代码的变动的前提下增加额外的功能。装饰器的返回值也是一个函数的对象，他经常用于有切实需求的场景。比如：插入日志、性能测试、事务处理、缓存、权限的校验等场景。有了装饰器就可以抽离出大量的与函数功能本身无关的雷同代码并发并继续使用。

> ### 生成器、迭代器的区别？

迭代器是一个更抽象的概念，任何对象，如果它的类有next方法和iter方法返回自己本身，对于string、zxssxlist、dict、tuple等这类容器对象，使用for循环遍历是很方便的。在后台for语句对容器对象调用iter()函数，iter()是Python的内置函数，iter()会返回一个定义了next()方法的迭代器对象，在没有后续元素时，next()会抛出一个StopIteration异常。

生成器(Generator)是创建迭代器的简单而强大的工具。它们写起来就像是正规的函数，只是在需要返回数据的时候使用yield语句。每次next()被调用时，生成器会返回它脱离的位置（它记忆语句最后一次执行的位置和所有的数据值）。

区别：生成器能做到迭代器能做的所有事，而且因为自动创建了iter()和next()方法，生成器显得特别简洁，而且生成器也是高效的，使用生成器表达式取代列表解析可以同时节省内存。除了创建和保存程序状态的自动方法，当发生器终结时，还会自动抛出StopIteration异常。

> ### Python中yield的用法？

yield就是保存当前程序执行状态。你用for循环的时候，每次取一个元素的时候就会计算一次。用yield的函数叫generator，和iterator一样，它的好处是不用一次计算所有元素，而是用一次计算一次，可以节省很多空间。Generator每次计算需要上一次计算结果，所以用yield，否则一return，上次计算结果就没有了。

> ### Python中的可变对象和不可变对象？

不可变对象，该对象所指向的内存中的值不能被改变，当改变某个变量的时候，由于其所指的值不能被改变，相当于把原来的值复制一份后再改变，这会开辟一个新的地址，变量再指向这个新的地址。

可变对象，该对象所指向的内存中的值可以被改变，变量（准确的说是引用）改变后，实际上是其所指的值直接发生改变，并没有发生复制行为，也没有开辟新的地址，通俗点说就是原地改变。

Python中，数值类型（int和float）、字符串str、元组tuple都是不可变类型。而列表list、字典dict、集合set是可变类型。

> ### Python中is和==的区别？

is判断的是a对象是否就是b对象，是通过id来判断的。

==判断的是a对象的值是否和b对象的值相等，是通过value来判断的。

> ### Python的魔法方法

魔法方法就是可以给你的类增加魔力的特殊方法，如果你的对象实现（重载）了这些方法中的某一个，那么这个方法就会在特殊的情况下被Python所调用，你可以定义自己想要的行为，而这一切都是自动发生的。它们经常是两个下划线包围来命名的（比如\__init__,\__It__)，Python的魔法方法是非常强大的，所以了解其使用方法也变的尤为重要！

\__init__构造器,当一个实例被创建的时候初始化的方法。但是它并不是实例化调用的第一个方法。

\__new__才是实例化对象调用的第一个方法，它只取下cls参数，并把其他参数传给\__init_。 \__new__很少使用，但是也有它适合的场景，尤其是当类继承自一个像元组或者字符串这样不经常改变的类型的时候。

\__call__允许一个类的实例像函数一样被调用。

\__getitem__定义获取容器中指定元素的行为，相当于self[key]。

\__getattr__定义当用户试图访问一个不存在属性的时候的行为。

\__setattr__定义当一个属性被设置的时候的行为。

\__getattribute__定义当一个属性被访问的时候的行为。

> ### 面向对象中怎么实现只读属性？

将对象私有化，通过共有方法提供一个读取数据的接口。

> ### 谈谈你对面向对象的理解？

面向对象是相对于面向过程而言的。面向过程语言是一种基于功能分析的、以算法为中心的程序设计方法；而面向对象是一种基于结构分析的、以数据为中心的程序设计思想。在面向对象语言中有一个很重要的东西，叫做类。

面向对象有三大特性：封装、继承、多态

Python里match与search的区别？

match()函数只检测RE是不是在string的开始位置匹配，

search()会扫描整个string查找匹配；

也就是说match()只有在0位置匹配成功的话才有返回，

如果不是开始位置匹配成功的话，match()就返回none。

> ### Python字符串查找和替换？

1. re.findall(r'目的字符串','原有字符串')  # 查询
2. re.findall[r'cast','itcast.cn'](0)
3. re.sub(r'要替换原字符','要替换新字符','原始字符串')
4. re.sub(r'cast','heima','itcast.cn')

> ### Python中的列表和元组有什么区别

列表是动态的，长度大小不固定，可以随意的增加、删除、修改元素

元组是静态的，长度在初始化的时候就已经确定不能更改，更无法增加、删除、修改元素

> ### 一句话解释什么样的语言能够用装饰器？

函数可以作为参数传递的语言，可以使用装饰器。

> ### 简述面向对象中\__new__和\__init__区别

```python
__init__是初始化方法，创建对象后，就立刻被默认调用了，可接受参数
1. __new__至少要有一个参数cls，代表当前类，此参数在实例化时由Python解释器自动识别。
2. __new__必须要有返回值，返回实例化出来的实例，这点在自己实现__new__时要特别注意，可以return父类(通过super(当前类名,cls))__new__出来的实例，或者直接是object的__new__出来的实例。
3. __init__有一个参数self，就是这个__new__返回的实例，__init__在__new__的基础上可以完成一些其它初始化的动作，__init__不需要返回值。
4. 如果__new__创建的是当前类的实例，会自动调用__init__函数，通过return语句里面调用的__new__函数的第一个参数是cls来保证是当前类实例，如果是其它类的类名；那么实际创建返回的就是其它类的实例，其实就不会调用当前类的__init__函数，也不会调用其它类的__init__函数。
```

> ### Python中生成随机整数、随机小数、0-1之间小数方法

随机整数：random.randint(a,b)，生成区间内的整数。

随机小数：习惯用numpy库，利用np.random.randn(5)生成5个随机小数。

0-1随机小数：random.random()，括号中不传参。

> ### Python中断言方法举例

assert()方法，断言成功，则程序继续执行，断言失败，则程序报错。

> ### 简述cookie和session的区别

1. session在服务器端，cookie在客户端（浏览器）。
2. session的运行依赖session id，而session id是存在cookie中的，也就是说，如果浏览器禁用了cookie，同时session也会失效，存储session时，健与cookie中的session id相同，值是开发人员设置的键值对信息，进行了base64编码，过期时间由开发人员设置。
3. cookie安全性比session差。

> ### 简述any()和all()方法

any()：只要迭代器中有一个元素为真就为真

all()：迭代器中所有的判断项返回都是真，结果才为真

> ### 深浅拷贝

浅拷贝copy有两种情况：

第一种情况：复制的对象中无复杂子对象，原来值的改变并不会影响浅复制的值，同时浅复制的值改变也并不会影响原来的值。原来值的id值与浅复制原来的值不同。

第二种情况：复制的对象中有复杂子对象（例如列表中的一个子元素是一个列表），改变原来的值中的复杂子对象的值，会影响浅复制的值。

深拷贝deepcopy：完全复制独立，包括内层列表和字典。

> ### 请将[i for i in range(3)]改成生成器

生成器是特殊的迭代器：

1. 列表表达式的[]改为()即可变成生成器。
2. 函数在返回值的时候出现yield就变成生成器，而不是函数了，中括号换成小括号即可

> ### 举例说明SQL注入和解决办法

当以字符串格式化书写方式的时候，如果用户输入的有;+SQL语句，后面的SQL语句会执行，比如例子中的SQL注入会删除数据库demo。

> ### Python传参数是传值还是传址？

Python中函数参数是引用传递（注意不是值传递）。对于不可变类型（数值型、字符串、元组），因变量不能修改，所以运算不会影响到变量自身；而对于可变类型（列表字典）来说，函数体运算可能会更改传入的参数变量。

> ### 什么是Python迭代器？

迭代器是可以遍历或迭代的对象。

> ### Python中的生成器是什么？

返回可迭代项集的函数称为生成器。

> ### 当Python退出时，为什么不清除所有分配的内存？

当Python退出时，尤其是那些对其它对象具有循环引用的Python模块或者从全局名称空间引用的对象并没有被解除分配或释放。

无法解除分配C库保留的那些内存部分。

退出时，由于拥有自己的高效清理机制，Python会尝试取消分配/销毁其他所有对象。

> ### 深拷贝和浅拷贝有什么区别？

在创建新实例类型时使用浅拷贝，并保留在新实例中复制的值。浅拷贝用于复制引用指针，就像复制值一样。这些引用指向原始对象，并且在类的任何成员中所做的更改也将影响它的原始副本。浅拷贝允许更快地执行程序，它取决于所使用的数据的大小。

深拷贝用于存储已复制的值。深拷贝不会将引用指针复制到对象。它引用一个对象，并存储一些其他对象指向的新对象。原始副本中所做的更改不会影响使用该对象的任何其他副本。由于为每个被调用的对象创建了某些副本，因此深拷贝会使程序的执行速度变慢。

> ### Python中的self是什么？

self是类的实例或对象。在Python中，self包含在第一个参数中。但是，Java中的情况并非如此，它是可选的。它有助于区分具有局部变量的类的方法和属性。init方法中的self变量引用新创建的对象，而在其他方法中，它引用其方法被调用的对象。

> ### Python中的局部变量和全局变量是什么？

全局变量：在函数外或全局空间中声明的变量称为全局变量。这些变量可以由程序中的任何函数访问。

局部变量：在函数内声明的任何变量都称为局部变量。此变量存在于局部空间中，而不是全局空间中。

> ###  如何在Python中管理内存？

Python中的内存管理由Python私有堆空间管理。所有Python对象和数据结构都位于私有堆中。程序员无权访问此私有堆。Python解释器负责处理这个问题。Python对象的堆空间分配由Python的内存管理器完成。核心API提供了一些程序员编写代码的工具。

Python还有一个内置的垃圾收集器，它可以回收所有未使用的内存，并使其可用于堆空间。

> ### 装饰器的写法以及应用场景

```python
含义：装饰器本质就是函数，为其他函数添加附加功能
原则：
不修改被修饰函数的代码
不修改被修饰函数的调用方式
应用场景：
无参装饰器在用户登录 认证中常见
有参装饰器在flask的路由系统中见到过

import functools
def wrapper(func):
@functools.wraps(func)
def inner(*args,**kwargs):
print('我是装饰器')
return func
return inner
@wrapper
def index():
print('我是被装饰函数')
return None
index()
# 应用场景
- 高阶函数
- 闭包
- 装饰器
- functools.wraps(func)
```

> ### Python垃圾回收机制

引用计数，标记清除，分代回收

1. 引用计数

​ 引用计数是用来记录对象被引用的次数，每当对象被创建或者被引用时将该对象的引用次数加一，当对象的引用被销毁时该对象的引用次数减一，当对象的引用次数减到零时，那么其所占用的空间也就可以被释放了。

2. 标记清除

​ 标记清除算法是一种基于追踪回收技术实现的垃圾回收算法。它分为两个阶段：第一阶段是标记阶段，第二阶段是把那些没有标记的对象非活动对象进行回收。

3. 分代回收

​ 将系统中的全部内存块依据其存活时间划分为不同的集合，每个集合就成为一个"代"，垃圾收集的频率随着“代”的存活时间的增大而减小。

> ### Python写原生sql

```sql
# 打开数据库连接 db =
pymysql.connect("localhost","testuser","test123","TESTDB")
# 使用cursor()方法获取操作游标 cursor = db.cursor()
# SQL删除语句  sql = "DELETE FROM EMPLOYEE WHERE AGE > %S" % (20) try:
# 执行SQL语句 cursor.execute(sql)
# 提交修改  db.commit() except:
# 发生错误时回滚 db.rollback()
# 关闭连接 db.close()
```

### MySQL常见数据库引擎及比较

#### InnoDB

支持事务

支持表锁、行锁 (for update)

表锁：select * from tb for update

行锁：select id,name from tb where id=2 for update

#### myisam

查询速度快

全文索引

支持表锁

表锁：select * from tb for update\

#### NDB

高可用、高性能、高可扩展性的数据库集群系统

#### Memory

默认使用的是哈希索引

### 什么是事务？MySQL如何支持事务？

事务用于将某些操作的多个SQL作为原子性操作，一旦有某一个出现错误，即可回滚到原来的状态，从而保证数据库数据完整性。

事务的特性：

原子性：确保工作单元内的所有操作都成功完成，否则事务将被中止在故障点，和以前的操作将回滚到以前的状态。

一致性：确保数据库正确地改变状态后，成功提交的事务。

隔离性：使事务操做彼此独立的和透明的。

持久性：确保提交的事物的结果或效果的系统出现故障的情况下仍然存在。

MySQL实现事务

InnoDB支持事务，MyISAM不支持

```sql
# 启动事务：
# start transaction；
# update from account set money = money-100 where name='a';
# update from account set money = money+100 where name='b';
# commit;
'start transaction 手动开启事务， commit 手动关闭事务'
```

### MySQL索引类型

1. 普通索引：是最基本的索引，它没有任何限制。
2. 唯一索引：与前面的普通索引类似，不同的是：索引列的值必须唯一，但允许有空值。

3. 主键索引：是一种特殊的唯一索引，一个表只能有一个主键，不允许有空值。
4. 组合索引：指多个字段上创建的索引，只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用。使用组合索引时遵循最左前缀集合。
5. 全文索引：注意用来查找文本中的关键字，而不是直接与索引中的值相比较。

​ fulltext索引跟其他索引大不相同，它更像是一个搜索引擎，而不是简单的where语句的参数匹配。

### 主键和外键的区别

主键：是能确定一条记录的唯一标示。例如，身份证号码

外键：用于与另一张表的关联，是能确定另一张表记录的字段，用于保持数据的一致性

#### 如何开启慢日志查询

```sql
修改配置文件
slow_query_log = OFF  是否开启慢日志记录 -75 -
long_query_time = 2  时间限制，超过此时间，则记录
slow_query_log_file = /usr/slow.log  日志文件
log_queries_not_using_indexes = OFF  为使用索引的搜索是否记录
下面是开启
slow_query_log = ON
long_query_time = 2
log_querise_not_using_indexes = OFF
log_queries_not_using_indexes = ON
注：查看当前配置信息：
show variables like '%query%'  修改当前配置
set global 变量名 = 值
```

### 数据库优化方案

1. 优化索引、SQL语句、分析慢查询；
2. 设计表的时候严格根据数据库的设计范式来设计数据库；
3. 使用缓存，把经常访问到的数据而且不需要经常变化的数据放在缓存中，能节约磁盘IO
4. 优化硬件；采用SSD，使用磁盘队列技术(RAID0,RAID1,RDID5)等
5. 采用MySQL内部自带的表分区技术，把数据分层不同的文件，能够提高磁盘的读取效率；
6. 垂直分表；把一些不经常读的数据放在一张表里，节约磁盘I/O；
7. 主从分离读写；采用主从复制把数据库的读操作和写入操作分离开来；
8. 分库分表分机器（数据量特别大），主要的原理就是数据路由；
9. 选择合适的表引擎，参数上的优化；
10. 进行架构级别的缓存，静态化和分布式；
11. 不采用全文索引；
12. 采用更快的存储方式，例如NoSQL存储经常访问的数据**。

### 事务的特性

1. 原子性(Atomicity)：事务中的全部操作在数据库中是不可分割的，要么全部完成，要么均不执行。
2. 一致性(Consistency)：几个并行执行的事务，其执行结果必须与按某一顺序串行执行的结果相一致。
3. 隔离性(Isolation)：事务的执行不受其他事务的干扰，事务执行的中间结果对其他事务必须是透明的。
4. 持久性(Durability)：对于任意已提交事务，系统必须保证该事务对数据库的改变不被丢失，即使数据库出现故障

### 数据库索引原理

数据库索引，是数据库管理系统中一个排序的数据结构，以协助快速查询、更新数据库表中数据。

索引的实现通常使用 B_TREE。 B_TREE索引加速了数据访问，因为存储引擎不会再去扫描整张表得到需要的数据；相反，它从根节点开始，根节点保存了子节点的指针，存储引擎会根据指针快速寻找数据。

### 数据库怎么优化查询效率？

1. 储存引擎选择：如果数据表需要事务处理，应该考虑使用InnoDB，因为它完全符合ACID特性。如果不需要事务处理，使用默认存储引擎MyISAM是比较明智的。
2. 分表分库，主从。
3. 对查询进行优化，要尽量避免全表扫描，首先应考虑where及order by涉及的列上建立索引
4. 应尽量避免在where子句中对字段进行null值判断，否则将导致引擎放弃使用索引而进行全表扫描
5. 应尽量避免在where子句中使用!=或<>操作符，否则将导致引擎放弃使用索引而进行全表扫描
6. 应尽量避免在where子句中使用or来连接条件，如果一个字段有索引，一个字段没有索引，将导致引擎放弃使用索引而进行全表扫描
7. Update语句，如果只更改1、 2个字段，不要update全部字段，否则频繁调用会引起明显的性能消耗，同时带来大量日志
8. 对于多张大数据量（这里几百条就算大了）的表JOIN，要分页再JOIN，否则逻辑读会很高，性能很差。

### MySQL集群的优缺点？

优点：

99.999%的高可用性

快速的自动失效切换

灵活的分布式体系结构，没有单点故障

高吞吐量和低延迟

可扩展性强，支持在线扩容

缺点：

存在很大限制，比如：不支持外键

部署、管理、配置很复杂

占用磁盘空间大、内存大

备份和恢复不方便

重启的时候，数据节点将数据load到内存需要很长的时间

### MySQL数据库如何分区、分表？

分表可以通过三种方式：MySQL集群、自定义规则和merge存储引擎。

分区有四类：

RANGE分区：基于属于一个给定连续区间的列值，把多行分配给分区。

LIST分区：类似于按RANGE分区，区别在于LIST分区是基于列值匹配一个离散值集合中的某个值来进行选择。

HASH分区：基于用户定义的表达式的返回值来进行选择的分区，该表达式使用将要插入到表中的这些行的列值进行计算。这个函数可以包含MySQL中有效的、产生非负整数值的任何表达式。

KEY分区：类似于按HASH分区，区别在于KEY分区只支持计算一列或多列，且MySQL服务器提供其自身的哈希函数。必须有一列或多列包含整数值。

### 如何对查询命令进行优化？

```sql
a. 应尽量避免全表扫描，首先应考虑where及order by 涉及的列上建立索引。
b. 应尽量避免在where子句中对字段进行null值判断，避免使用!=或<>操作符，避免使用or连接条件，或在where子句中使用参数、对字段进行表达式或函数操作，否则会导致权标扫描
c. 不要在where子句中的"="左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引。
d. 使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用。
e. 很多时候可考虑用exists代替in。
f. 尽量使用数字型字段。
g. 尽可能的使用varchar/nvarchar代替char/nchar。
h. 任何地方都不要使用select from t，用具体的字段列表代替""，不要返回用不到的任何字段。
i. 尽量使用表变量来代替临时表。
j. 避免频繁创建和删除临时表，以减少系统表资源的消耗。
k. 尽量避免使用游标，因为游标的效率较差。
l. 在所有的存储过程和触发器的开始处设置SET NOCOUNT ON，在结束时设置SET NOCOUNT OFF。
m. 尽量避免大事务操作，提供系统并发能力。
n. 尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理。
```

### Sql注入是如何产生的，如何防止？

程序开发过程中不注意规范书写sql语句和对特殊字符进行过滤，导致客户端可以通过全局变量POST和GET提交一些sql语句正常执行。产生Sql注入。下面是防止办法：

```sql
a. 过滤掉一些常见的数据库操作关键字，或者通过系统函数来进行过滤。
b. 在PHP配置文件中奖Register_globals=off;设置为关闭状态
c. SQL语句书写的时候尽量不要省略小引号（tab健上面那个）和单引号
d. 提高数据库命名技巧，对于一些重要的字段根据程序的特点命名，取不易被猜到的
e. 对于常用的方法加以封装，避免直接暴露SQL语句
f. 开启PHP安全模式：Safe_mode=on;
g. 打开magic_quotes_gpc来防止SQL注入
h. 控制错误信息：关闭错误提示信息，将错误信息写到系统日志。
i. 使用mysqli或pdo预处理
```

### NoSQL和关系数据库的区别？

```sql
a. SQL数据存在特定结构的表中；而NoSQL则更加灵活和可扩展，存储方式可以是JSON文档、哈希表或者其他方式。
b. 在SQL中，必须定义好表和字段结构后才能添加数据，例如定义表的主键(primary key)，索引(index)、触发器(trigger)，存储过程(stored procedure)等。表结构可以在被定义之后更新，但是如果有比较大的结构变更的话就会变得比较复杂。在NoSQL中，数据可以在任何时候任何地方添加，不需要先定义表。
c. SQL中如果需要增加外部关联数据的话，规范化做法是在原表中增加一个外键，关联外部数据表。而在NoSQL中除了这种规范化的外部数据表做法以外，我们还能用如下的非规范化方式把外部数据直接放到原数据集中，以提高查询效率。缺点也比较明显，更新审核人数据的时候将会比较麻烦。
d. SQL中可以用JOIN表链接方式将多个关系数据表中的数据用一条简单的查询语句查询出来。NoSQL暂未提供类似JOIN的查询方式对多个数据集中的数据做查询。所以大部分NoSQL使用非规范化的数据存储方式存储数据。
e. SQL中不允许删除已经被使用的外部数据，而NoSQL中则没有这种强耦合的概念，可以随时删除任何数据。
f. SQL中如果多张表数据需要同批次被更新，即如果其中一张表更新失败的话，其他表也不能更新成功。这种场景可以通过事务来控制，可以在所有命令完成后再统一提交事务。而NoSQL中没有事务这个概念，每一个数据集的操作都是原子级的。
g. 在相同水平的系统设计的前提下，因为NoSQL中省略了JOIN查询的消耗，故理论上性能上是优于SQL的。
```

### 存储过程和函数的区别？

相同点：存储过程和函数都是为了可重复的执行操作数据库的sql语句的集合。

1. 存储过程和函数都是一次编译，就会被缓存起来，下次使用就直接命中已经编译好的sql语句，不需要重复使用。减少网络交互，减少网络访问流量。

不同点：标识符不同，函数的标识符是function，存储过程是proceduce。

1. 函数中有返回值，且必须有返回值，而过程没有返回值，但是可以通过设置参数类型(in,out)来实现多个参数或者返回值。
2. 存储函数使用select调用，存储过程需要使用call调用。
3. select语句可以在存储过程中调用，但是除了select..into之外的select语句都不能在函数中使用。
4. 通过in out 参数，过程相关函数更加灵活，可以返回多个结果。

### 数据库的设计？

第一范式：数据库表的每一列都是不可分割的原子数据项，即列不可拆分。

第二范式：建立在第一范式的基础上，要求数据库表中的每个实例或记录必须是可以唯一被区分的，即唯一标识。

第三范式：建立在第二范式的基础上，任何非主属性不依赖与其他非主属性，即引用主键。

### MySQL日志有哪些

错误日志：记录启动，运行或者停止mysql时出现的问题；

通用日志：记录建立的客户端连接和执行的语句；

二进制日志：记录所有更改数据的语句；

慢查询日志：记录所有执行时间超过long_query_time秒的查询或者不适用索引的查询。通过使用--slow_query_log[={0|1}]选项来启用慢查询日志，所有执行时间超多long_query_time的语句都会被记录。

### MySQL的常见命令

1. create database name;  创建数据库
2. use databasename;  选择数据库
3. drop database name;  直接删除数据库，不提醒
4. show tables;  显示表
5. describe tablename;  表的详细描述
6. select 中加上 distinct 去除重复字段
7. mysqladmin drop databasename  删除数据库前，有提示。
8. select version(),current_date;  显示当前mysql版本和当前日期

### 说一下MySQL数据库存储的原理

储存过程是一个可编程的函数，它在数据库中创建并保存。它可以有SQL语句和一些特殊的控制结构组成。当希望在不同的应用程序或平台上执行相同的函数，或者封装特定功能时，存储过程是非常有用的。数据库中的存储过程可以看做是对编程中面向对象方法的模拟。它允许控制数据的访问方式。存储过程通常有以下优点：

1. 存储过程能实现较快的执行速度
2. 存储过程允许标准组件是编程
3. 存储过程可以用流程控制语句编写，有很强的灵活性，可以完成复杂的判断和较复杂的运算。
4. 存储过程可被作为一种安全机制来充分利用
5. 存储过程能够减少网络流量

MySQL主从读写分离

当数据库的写压力增加，cache层（如Memcached）只能缓解数据库的读取压力。读写集中在一个数据库上让数据库不堪重负。使用主从复制技术（master-slave 模式）来达到读写分离，以提高读写性能和读库的可扩展性。读写分离就是只在主服务器上写，只在从服务器上读，基本原理是让主数据库处理事务性查询，而从数据库处理select查询，数据库复制被用于把事务性查询（增删改）导致的改变更新同步到集群中的从数据库。

MySQL读写分离提升系统性能：

1. 主从只负责各自的读和写，极大程度缓解X锁和S锁争用
2. slave可以配置MyISAM引擎，提升查询性能以及节约系统开销
3. master直接写是并发的，slave通过主库发送来的binlog恢复数据是异步的
4. slave可以单独设置一些参数来提升其读的性能
5. 增加冗余，提高可用性

实现主从分离可以使用MySQL中间件如：Atlas

### 分表分库

在cache层的高速缓存，MySQL的主从复制，读写分离的基础上，这时MySQL主库的写压力开始出现瓶颈，而数据量的持续猛增，由于MyISAM使用表锁，在高并发下会出现严重的锁问题，大量的高并发MySQL应用开始使用InnoDB引擎带起MyISAM。采用Master-Slave复制模式的MySQL架构，只能对数据库的读进行扩展。而对数据的写操作还是集中在Master上。这时需要对数据库的吞吐能力进一步地扩展，以满足高并发访问与海量数据存储的需求。对于访问极为频繁且数据量巨大的单表来说，首先要做的是减少单表的记录条数，以便减少数据查询所需的时间，提供数据库的吞吐，这就是所谓的分表【水平拆分】。在分表之前，首先需要选择适当的分表策略（尽量避免分出来的多表关联查询），使得数据能够较为均衡地分布到多张表中，并且不影响正常的查询。分表能够解决单表数据量过大带来的查询效率下降的问题，但是却无法给数据库的并发处理能力带来质的提升。面对高并发的读写访问，当数据库master服务器无法承载写操作压力时，不管如何扩展Slave服务器都是没有意义的，对数据库进行拆分，从而提高数据库写入能力，即分库【垂直拆分】。

### 负载均衡集群

将大量的并发请求分担到多个处理节点。由于单个处理节点的故障不影响整个服务，负载均衡集群同时也实现了高可用性。负载均衡将是大型网站解决高负荷访问和大量并发请求采用的终极解决办法。

### MySQL有哪些锁

#### 加锁机制

1. 乐观锁：先修改，保存时判断是否被更新过，应用级别
2. 悲观锁：先获取锁，再操作修改，数据库级别

#### 锁粒度

表级锁：开销小，加锁快，粒度大，锁冲突概率大，并发度低，适用于读多写少的情况。

页级锁：BDB存储引擎

行级锁：Innodb存储引擎，默认选项

#### 兼容性

S锁，也叫做读锁、共享锁，对应于我们常用的select * from users where id = 1 lock in share mode

X锁，也叫做写锁、排它锁、独占锁、互斥锁，对应于select * from users where id = 1 for update

下面这个表格是锁冲突矩阵，可以看到只有读锁和读锁之间兼容的，写锁和读锁、写锁都是冲突的。

#### 共享锁和排斥锁

冲突的时候会阻塞当前会话，直到拿到锁或者超时

这里要提到的一点是，S锁和X锁是可以是表锁，也可以是行锁

接下来是面试必备的。

记录锁：单行记录上的锁，行锁是加在索引上的。

间隙锁：锁定记录之间的范围，但不包含记录本身。

Next Key Lock：记录锁 + 间隙锁，锁定一个范围，包含记录本身。

#### 意向锁(Intention Locks)

InnoDB为了支持多粒度（表锁与行锁）的锁并存，引入意向锁。意向锁是表级锁，

IS：意向共享锁

IX：意向排他锁

事务在请求某一行的S锁和X锁前，需要先获得对应表的IS、IX锁。

意向锁产生的主要目的是为了处理行锁和表锁之间的冲突，用于表明“某个事务正在某一行上持有了锁，或者准备去持有锁”。比如，表中的某一行上加了X锁，就不能对这张表加X锁。

如果不在表上加意向锁，对表加锁的时候，都要去检查表中的某一行上是否加有行锁，多麻烦。

#### 插入意向锁（Insert Intention Lock）

Gap Lock中存在一种插入意向锁，在insert操作时产生。

有两个作用：

和next-key互斥，阻塞next-key锁，防止插入数据，这样就不会幻读。

插入意向锁互相是兼容的，允许相同间隙、不同数据的并发插入

### 为什么要加锁

数据库是一个多用户使用的共享资源。当多个用户并发地存取数据时，在数据库中加锁是为了保证数据库的一致性。

数据库有ACID原则，其中I是隔离性，

- 脏读：读未提交的数据
- 不可重复度：读已修改的数据
- 虚读：读提交了插入/删除的数据

### 锁总结

表锁其实我们程序员是很少关心它的：

在MyISAM存储引擎中，当执行SQL语句的时候是自动加的。

在InnoDB存储引擎中，如果没有使用索引，表锁也是自动加的。

现在我们大多数使用MySQL都是使用InnoDB，InnoDB支持行锁：

共享锁--->读锁--->S锁

排他锁--->写锁--->X锁

在默认的情况下，select是不加任何行锁的，事务可以通过以下语句显示给记录集加共享锁或排他锁。

### MySQL主从搭建原理，以及如何监控

原理：

从库生成两个线程，一个I/O线程，一个SQL线程；

I/O线程去请求主库的binlog，并将得到的binlog日志写到relay log(中继日志)

文件中：

主库会生成一个log dump线程，用来给从库I/O线程传binlog；

SQL线程，会读取relay log 文件中的日志，并解析成具体操作，来实现主从的操作一致，而最终数据一致

监控：

首先要切割出来双Yes和延时时间不能超过120，然后再判断。

### 简述数据库分库分表

1. 分库

​ 当数据库中的表太多，将某些表分到不同数据库，例如：1W张表时代价：连表查询跨数据库，代码变多

2. 分表

​ 水平分表：将某些列拆分到另一张表，例如：博客+博客详情

​ 垂直分表：将某些历史信息，分到另外一张表中，例如：支付宝账单

### 什么是索引合并

1. 索引合并是把几个索引的范围扫描合并成一个索引。
2. 索引合并的时候，会对索引进行并集，交集或者先交集再并集操作，以便合并成一个索引。
3. 这些需要合并的索引只能是一个表的。不能对多表进行索引合并。简单的说，索引合并，让一条SQL可以使用多个索引。对这些索引取交集，并集，或者先取交集再取并集。从而减少从数据表中取数据的次数，提高查询效率。

### 什么是覆盖索引

在索引表中就能将想要的数据查询到

### 简述数据库读写分离

- 实现：两台服务器同步数据
- 利用数据库的主从分离：主，用于删除、修改、更新；从，用于查；

```sql
方式一：是视图里面用using方式可以进行指定到哪个数据读写

from django.shortcuts import render,HttpResponse

from app01 import models

def index(request):

models.UserType.objects.using('db1').create(title='普通用户')

#手动指定去某个数据库取数据

result = models.UserType.objects.all().using('db1')
print(result)
return HttpResponse('...')
方式二：写配置文件
class Router1:
# 指定到某个数据库取数据
def db_for_read(self,model,**hints):
"""
Attempts to read auth models go to auth_db.
"""
if model._meta.model_name == 'usertype';
return 'db1'
else:
return 'default'
# 指定到某个数据库存数据
def db_for_write(self,model,**hints):
"""
Attempts to write auth models go to auth_db.
"""
return 'default'  再写到配置
DATABASES = {
'default': {
'ENGINE': 'django.db.backends.sqlite3',
'NAME': os.path.join(BASE_DIR,'db.sqlite3'),
},
'db1': {
'ENGINE': 'django.db.backends.sqlite3',
'NAME': os.path.join(BASE_DIR,'db.sqlite3'),
}
}
DATABASE_ROUTERS = ['db_router.Router1',]
```

### 什么是大事务以及如何处理

运行时间比较长，操作的数据比较多的事务称为大事务

1. 锁定太多的数据，造成大量的阻塞和锁超时
2. 回滚时所需时间比较长
3. 执行时间长，容易造成主从延迟

如何处理大事务

1. 避免一次处理太多的数据
2. 移出不必要在事务中的SELECT操作

### MySQL的四种隔离级别

#### Read Uncommitted (读取未提交内容)

在该隔离级别，所有事务都可以看到其它未提交事务的执行结果。本隔离级别很少用于实际应用，因为它的性能也不比其它级别好多少。读取未提交的数据，也别称之为脏读（Dirty Read）。

#### Read Committed（读取提交内容）

这是大多数数据库系统的默认隔离级别（但不是MySQL默认的）。它满足了隔离的简单定义：一个事务只能看见已经提交事务所做的改变。这种隔离级别也支持所谓的不可重复读（Nonrepeatable Read），因为同一事务的其他实例在该实例处理其间可能会有新的commit，所以同一select可能返回不同结果。

#### Repeatable Read（可重读）

这是MySQL的默认事务隔离级别，它确保同一事务的多个实例在并发读取数据时，会看到同样的数据行。不过理论上，这会导致另一个棘手的问题：幻读（Phantom Read）。简单的说，幻读指当用户读取某一范围的数据行时，另一事务又在该范围内插入了新行，当用户再读取该范围的数据行时，会发现有新的“幻影”行。InnoDB和Falcon存储引擎通过多版本并发控制（MVCC，Multiversion Concurrency Control）机制解决了该问题。

#### Serializable（可串行化）

这是最高的隔离级别，它通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁。在这个级别，可能导致大量的超时现象和锁竞争。这四种隔离级别采取不同的锁类型就容易发生问题。例如：

#### 脏读（Drity Read）

某个事务已更新一份数据，另一个事务在此时读取了同一份数据，由于某些原因，前一个RollBack了操作，则后一个事务所读取的数据就会是不正确的。

#### 不可重复读(Non-repeatable read)

在一个事务的两次查询之中数据不一致，这可能是两次查询过程中间插入了一个事务更新的原有的数据。

#### 幻读(Phantom Read)

在一个事务的两次查询中数据笔数不一致，例如有一个事务查询了几列(Row)数据，而另一个事务却在此时插入了新的几列数据，先前的事务在接下来的查询中，就有几列数据是未查询出来的，如果此时插入和另外一个事务插入的数据，就会报错。

### MySQL的常见命令如下

```sql
1. create database name;  创建数据库
2. use databasename;  选择数据库
3. drop database name 直接删除数据库，不提醒
4. show tables;  显示表
5. describe tablename;  表的详细描述
6. select 中加上 distinct 去除重复字段
7. mysqladmin drop databasename  删除数据库前，有提示。
8. select version(),current_date;  显示当前mysql版本和当前日期
```

### SQL的select语句完整的执行顺序

#### SQL Select 语句完整的执行顺序

1. from 子句组装来自不同数据源的数据；
2. where 子句基于指定的条件对记录行进行筛选；
3. group by 子句将数据划分为多个分组；
4. 使用聚集函数进行计算；
5. 使用having子句筛选分组；
6. 计算所有的表达式；
7. select的字段；
8. 使用order by对结果集进行排序。

SQL语言不同于其他编程语言的最明显特征是处理代码的顺序。在大多数据库语言中，代码按编码顺序被处理。但在SQL语句中，第一个被处理的子句式FROM，而不是第一出现的SELECT。

#### SQL查询处理的步骤序号

```sql
(1) FROM<left_table>
(2) <join_type>JOIN<right_table>
(3) ON<join_condition>
(4) WHERE<where_condition>
(5) GROUP BY<group_by_list>
(6) WITH{CUBE | ROLLUP}
(7) HAVING<having_condition>
(8) SELECT
(9) DISTINCT
(10) ORDER BY <order_by_list>
(11) <TOP_specification> <select_list>
```

以上每个步骤都会产生一个虚拟表，该虚拟表被用作下一个步骤的输入。这些虚拟表对调用者（客户端应用程序或者外部查询）不可用。只有最后一步生成的表才会给调用者。如果没有在查询中指定某一个子句，将跳过相应的步骤。

#### 逻辑查询处理阶段简介

1. FROM：对FROM子句中的前两个表执行笛卡尔积（交叉联接），生成虚拟表VT1。
1. ON：对VT1应用ON筛选器，只有那些使为真才被插入到TV2。
1. OUTER(JOIN)：如果指定了OUTER JOIN(相对于CROSS JOIN 或 INNER JOIN)，保留表中未找到匹配的行将作为外部行添加到VT2，生成TV3。如果FROM子句包含两个以上的表，则对上一个联接生成的结果表和下一个表重复执行步骤1到步骤3，直到处理完所有的表位置。
1. WHERE：对TV3应用WHERE筛选器，只有使为true的行才插入TV4。
1. GROUP BY：按GROUP BY子句中的列列表对TV4中的行进行分组，生成TV5。
1. CUTE|ROLLUP：把超组插入VT5，生成VT6。
1. HAVING：对VT6应用HAVING筛选器，只有使为true的组插入到VT7。
1. SELECT：处理SELECT列表，产生VT8。
1. DISTINCT：将重复的行从VT8中删除，产品VT9。
1. ORDER BY：将VT9中的行按ORDER BY子句中的列列表顺序，生成一个游标(VC 10)。
1. TOP：从VC10的开始处选择指定数量或比例的行，生成表TV11，并返回给调用者。where子句中的条件书写顺序。
