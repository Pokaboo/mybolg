## Redis

### 什么是NoSQL
- Not Only Sql

- 高性能读取

- 高可用

- 存储数据、缓存

- 水平（横向）扩展方便高效

- MemCache、Redis、MongoDB被称为NoSQL三剑客。那么时代为什么需要NoSQL数据库呢？我们来做个对比：

  |              | 关系型数据库             | NoSQL数据库                            |
  | ------------ | ------------------------ | -------------------------------------- |
  | 数据存储位置 | 硬盘                     | 内存                                   |
  | 数据结构     | 高度组织化结构化数据     | 没有预定义的模式                       |
  | 数据操作方式 | SQL                      | 所有数据都是键值对，没有声明性查询语言 |
  | 事务控制     | 严格的基础事务ACID原则   | 基于乐观锁的松散事务控制               |
  | 访问控制     | 细粒度的用户访问权限控制 | 简单的基于IP绑定或密码的访问控制       |
  | 外键         | 支持                     | 不支持                                 |
  | 索引         | 支持                     | 不支持                                 |

> 所以NoSQL数据库的最大优势体现为：高性能、高可用性和可伸缩性。

### 分布式缓存

- 提升读取速度性能
- 分布式计算领域
- 降低数据库查询压力
- 跨服务器缓存
- 内存式缓存

### Redis是什么
- Redis是现在最受欢迎的NoSQL数据库之一，Redis是一个使用ANSI C编写的开源、包含多种数据结构、支持网络、基于内存、可选持久性的键值对存储数据库，其具备如下特性：
    - 基于内存运行，性能高效
    - 支支持分布式，理论上可以无限扩展
    - key-value存储系统
    - 开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API
- Redis 的应用场景包括：缓存系统（“热点”数据：高频读、低频写）、计数器、消息队列系统、排行榜、社交网络和实时系统
- Redis提供的数据类型主要分为5种自有类型和一种自定义类型，这5种自有类型包括：String类型、哈希类型、列表类型、集合类型和顺序集合类型。

### Redis的应用场景

- Redis的典型应用场景
  - 缓存：使用Redis可以建立性能非常出色的缓存服务器，查询请求先在Redis中查找所需要的数据，如果能够查询到（命中）则直接返回，大大减轻关系型数据库的压力。
  - 数据临时存储位置：使用token（令牌）作为用户登录系统时的身份标识，这个token就可以在Redis中临时存储。
  - 分布式环境下解决Session不一致问题时的Session库：Spring提供了一种技术解决分布式环境下Session不一致问题，叫SpringSession。而Redis就可以为SpringSession提供一个数据存储空间。
  - 流式数据去重：在Redis中有一种数据类型是set，和Java中的Set集合很像，不允许存储重复数据。借助这个特性我们可以在Redis中使用set类型存储流式数据达到去重的目的。
- Redis不适用的场景
  - 直接查询value：Redis是一个严格的Key-value数据库，所有数据都必须通过key去找到value，Redis没有提供直接根据查询条件匹配value的方法。
  - 用多键一值表示特定关系：Redis中一个key对应一个value，没有多个key对应同一个value的情况。
  - 事务中回滚：Redis不支持回滚。如果一个命令在加入队列时没有检测出问题，那么队列执行时不会因为某一条命令失败而回滚。

### 缓存方案对比
- Ehcache
    - 优点：
        - 基于java开发 
        - 基于jvm
        - 简单、轻巧、方便
    - 缺点：
        - 集群不支持
        - 分布式不支持

- Mencache
    - 优点：
        - 简单的key-value存储
        - 内存使用率比较高
        - 多核处理，多线程
    - 缺点：
        - 无法容灾
        - 无法持久化

- Redis
    - 优点：
        - 丰富的数据结构
        - 持久化
        - 内存数据库
        - 主从同步，故障转移
    - 缺点：
        - 单线程
        - 单核 
    
###  阻塞与非阻塞
- IO的方式通常分为几种，同步阻塞的BIO、同步非阻塞的NIO、异步非阻塞的AIO。
    - BIO:同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。每一个线程守着一个IO通道，当没有IO时这个线程就阻塞着，一旦有IO事件发生，这个线程就开始工作
    - NIO:同步非阻塞，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。socket主要的读、写、注册和接受函数，在等待就绪阶段都是非阻塞的，真正的IO操作是同步阻塞的。将每个连接(IO通道)都注册到Selector多路复用器上，告诉复用器我已经连接了，如果有IO事件发生，你就通知我。Selector就不断地调用select函数，去访问每一个通道看那个通道有IO事件发生，如果发生了就通知，内核开启一个对应事件的线程去工作
    - AIO :异步非阻塞，服务器实现模式为一个有效请求一个线程，客户端的I/O请求都是由OS先完成了再通知服务器应用去启动线程进行处理。当进行读写操作时，只须直接调用API的read或write方法即可。这两种方法均为异步的，对于读操作而言，当有流可读取时，操作系统会将可读的流传入read方法的缓冲区，并通知应用程序；对于写操作而言，当操作系统将write方法传递的流写入完毕时，操作系统主动通知应用程序。 即可以理解为，read/write方法都是异步的，完成后会主动调用回调函数

### 多路复用器
- 为什么 Redis 中要使用 I/O 多路复用这种技术呢？

  ```
  首先，Redis 是跑在单线程中的，所有的操作都是按照顺序线性执行的，但是由于读写操作等待用户输入或输出都是阻塞的，所以 I/O 操作在一般情况下往往不能直接返回，这会导致某一文件的 I/O 阻塞导致整个进程无法对其它客户提供服务，而 I/O 多路复用就是为了解决这个问题而出现的
  
  ```


### Redis线程模型
[![cFdOGF.png](https://z3.ax1x.com/2021/03/30/cFdOGF.png)](https://imgtu.com/i/cFdOGF)


### Redis发布与订阅

### Redis持久化
- Redis的持久化机制RDB（Redis DataBase）
    - 1、什么是RDB：每隔一段时间，把内存中的数据写入磁盘的临时文件，作为快照，恢复的时候把快照文件读进内存。如果宕机重启，那么内存里的数据肯定会没有的，那redis后，则会恢复
    - 2、备份与回复
        - 内存备份--->磁盘临时文件
        - 临时文件--->恢复导内存
    - 3、RDB优劣势
        - 优势：
            - 1、每隔一段时间备份，全量备份
            - 2、灾备简单，可以远程传输
            - 3、子进程备份的时候，主进程不会有任何io操作（不会有写入修改或删除），保证备份数据的完整性
            - 相对于AOF来说，当有更大文件的时候可以快速重启恢复           
        - 劣势：
            - 1、 发生故障是，有可能会丢失最后一次的备份数据 
            - 2、子进程所占用的内存比会和父进程一模一样，如会造成CPU负担
            - 3、由于定时全量备份是重量级操作，所以对于实时备份，就无法处理了。
    
- Redis的持久化机制AOF（Append Only File）
  [![cMv8nH.png](https://z3.ax1x.com/2021/04/05/cMv8nH.png)](https://imgtu.com/i/cMv8nH)
    - 1、RDB会丢失最后一次备份的rdb文件，但是其实也无所谓，其实也可以忽略不计，毕竟是缓存，丢了就丢了，但是如果追求数据的完整性，那就的考虑使用AOF了
    - 2、AOF特点：
        - 1、以日志的形式来记录用户请求的写操作。读操作不会记录，因为写操作才会存存储。
        - 2、文件以追加的形式而不是修改的形式
        - 3、redis的aof恢复其实就是把追加的文件从开始到结尾读取执行写操作
    - 优劣势：
        - 优势：
            - 1、AOF更加耐用，可以以秒级别为单位备份，如果发生问题，也只会丢失最后一秒的数据，大大增加了可靠性和数据完整性。所以AOF可
            一次，使用fsync操作
            - 2、以log日志形式追加，如果磁盘满了，会执行 redis-check-aof 工具
            - 3、当数据太大的时候，redis可以在后台自动重写aof。当redis继续把日志追加到老的文件中去时，重写也是非常安全的，不会影响客户端
            作
            - 4、AOF 日志包含的所有写操作，会更加便于redis的解析恢复
        - 劣势：
            - 1、相同的数据，同一份数据，AOF比RDB大
            - 2、针对不同的同步机制，AOF会比RDB慢，因为AOF每秒都会备份做写操作，这样相对与RDB来说就略低。 每秒备份fsync没毛病，但是
            的每次写入就做一次备份fsync的话，那么redis的性能就会下降
            - 3、AOF发生过bug，就是数据恢复的时候数据不完整，这样显得AOF会比较脆弱，容易出现bug，因为AOF没有RDB那么简单，但是呢为
            的产生，AOF就不会根据旧的指令去重构，而是根据当时缓存中存在的数据指令去做重构，这样就更加健壮和可靠了
    - 到底采用RDB还是AOF呢？
        - 1、如果你能接受一段时间的缓存丢失，那么可以使用RDB
        - 2、如果你对实时性的数据比较care，那么就用AOF 

### 主从复制原理
- 主从复制（读写分离）
[![cMzPL4.png](https://z3.ax1x.com/2021/04/05/cMzPL4.png)](https://imgtu.com/i/cMzPL4)
- 主从模式

### Redis 缓存过期处理与内存淘汰机制
- 计算机内存有限，越大越贵，Redis的高并发高性能都是基于内存的，用硬盘的话GG
- 已过期的key如何处理：虽然key过期了，但是只要没有被redis清理，那么其实内存还是会被占用着的
    - 设置了expire的key缓存过期了，但是服务器的内存还是会被占用，这是因为redis所基于的两种删除策略，edis有两种策略：
        - 1. （主动）定时删除：定时随机的检查过期的key，如果过期则清理删除。（每秒检查次数在redis.conf中的hz配置）
        - 2. （被动）惰性删除：当客户端请求一个已经过期的key的时候，那么redis会检查这个key是否过期，如果过期了，则删除，然后返回一个nil。这种策略
        友好，不会有太多的损耗，但是内存占用会比较高。
- 那么如果内存被Redis缓存占用慢了咋办？redis提供了一套缓存淘汰机制：MEMORY MANAGEMENT

### Redis 的哨兵模式
- 问题：Master挂了，如何保证可用性，实现继续读写？
- 什么是哨兵：Sentinel(哨兵)是用于监控Redis集群中Master状态的工具，是 Redis 高可用解决方案，哨兵可以监视一个或者多个redis master服务，以及这些master服务的所有某个master服务宕机后，会把这个master下的某个从服务升级为master来替代已宕机的master继续工作。
- 原Master恢复后不同步问题：Master恢复成Slave后，他的同步状态不OK，状态为master_link_status:down ，这是为什么呢？
    - 1. 网络通信问题，要保证互相ping通，内网互通
    - 2. 关闭防火墙，对应的端口开发（虚拟机中建议永久关闭防火墙，云服务器的话需要保证内网互通）。
    - 3. 统一所有的密码，不要漏了某个节点没有设置。

### Redis 集群
- 1. 每个节点知道彼此之间的关系，也会知道自己的角色，当然他们也会知道自己存在与一个集群环境中，他们彼此之间可以交互和通信，ong。那么这些关系都会保存到某个配置文件中，每个节点都有，这个我们在搭建的时候会做配置的。
- 2. 客户端要和集群建立连接的话，只需要和其中一个建立关系就行。
- 3. 某个节点挂了，也是通过超过半数的节点来进行的检测，客观下线后主从切换，和我们之前在哨兵模式中提到的是一个道理。
- 4. Redis中存在很多的插槽，又可以称之为槽节点，用于存储数据，这个先不管，后面再说。

### 集群容错
- 构建Redis集群，需要至少3个节点作为master，以此组成一个高可用的集群，此外每个master都需要配备一个slave，所以整个集群需要6个节点，这也是最经典集群，也可以称之为三主三从，容错性更佳。

### 缓存穿透
- 缓存穿透：查询的key在redis中不存在，对应的id在数据库中也不存在，此时被非法的用户进行攻击，大量的请求会直接打在db上，造成宕机，从而影响整个系统，这种现象称之为缓存穿透
- 缓存穿透解决方案：
    - 方案一：把空的数据也缓存起来，比如空字符串，空对象，空数组，list集合等。
    - 方案二：布隆过滤器；
    [![cI8kPH.png](https://z3.ax1x.com/2021/04/18/cI8kPH.png)](https://imgtu.com/i/cI8kPH)

    [![cI8msP.png](https://z3.ax1x.com/2021/04/18/cI8msP.png)](https://imgtu.com/i/cI8msP)

### 缓存雪崩
- 原因:由于缓存层承载着大量请求，有效地保护了存储层，但是如果缓存层由于某些原因不可用（宕机）或者大量缓存由于超时时间相同在同一时间段失效（大批key失效/热点数据失效），大量请求直接到达存储层，存储层压力过大导致系统雪崩
- 解决方案：
    - ①、可以把缓存层设计成高可用的，即使个别节点、个别机器、甚至是机房宕掉，依然可以提供服务。利用sentinel或cluster实现
    - ②、采用多级缓存，本地进程作为一级缓存，redis作为二级缓存，不同级别的缓存设置的超时时间不同，即使某级缓存过期了，也有其他级别缓存兜底
    - ③、 缓存的过期时间用随机值，尽量让不同的key的过期时间不同（例如：定时任务新建大批量key，设置的过期时间相同）
- 预防:
    - ①、永不过期
    - ②、过期时间错开
    - ③、多缓存结合

### 缓存击穿
- 系统中存在以下两个问题时需要引起注意：
    - 当前key是一个热点key（例如一个秒杀活动），并发量非常大。
    - 重建缓存不能在短时间完成，可能是一个复杂计算，例如复杂的SQL、多次IO、多个依赖等。
- 在缓存失效的瞬间，有大量线程来重建缓存，造成后端负载加大，甚至可能会让应用崩溃。
- 解决方案：
    - ①、分布式互斥锁：只允许一个线程重建缓存，其他线程等待重建缓存的线程执行完，重新从缓存获取数据即可。set(key,value,timeout)
    - ②、永不过期
    - 2种方案对比：
        - 分布式互斥锁：这种方案思路比较简单，但是存在一定的隐患，如果在查询数据库 + 和 重建缓存（key失效后进行了大量的计算）时间过长，也可能会存在死锁和线程池阻塞的风险，高并发情景下吞吐量会大大降低！但是这种方法能够较好地降低后端存储负载，并在一致性上做得比较好
        - “永远不过期”：这种方案由于没有设置真正的过期时间，实际上已经不存在热点key产生的一系列危害，但是会存在数据不一致的情况，同时代码复杂度会增大。

### 常见面试题：

```
1.什么是 Redis?
2.Redis 的数据类型？
3.使用 Redis 有哪些好处？
4.Redis 相比 Memcached 有哪些优势？
5.Memcached 与 Redis 的区别都有哪些？
6.Redis 是单进程单线程的吗？为何它那么快那么高效？
7.一个字符串类型的值能存储最大容量是多少？
8.Redis 的持久化机制是什么？各自的优缺点？
9.Redis 常见性能问题和解决方案有哪些?
10.Redis 过期键的删除策略？
11.Redis 的回收策略（淘汰策略）?
12.为什么Redis 需要把所有数据放到内存中？
13.Redis 的同步机制了解么？
14.Pipeline 有什么好处，为什么要用 Pipeline？
15.是否使用过 Redis 集群，集群的原理是什么？
16.Redis 集群方案什么情况下会导致整个集群不可用？
17.Redis 支持的 Java 客户端都有哪些？官方推荐用哪个？
18.Jedis 与 Redisson 对比有什么优缺点？
19.Redis 如何设置密码及验证密码？
20.说说 Redis 哈希槽的概念？
21.Redis 集群的主从复制模型是怎样的？
22.Redis 集群会有写操作丢失吗？为什么？
23.Redis 集群之间是如何复制的？
24.Redis 集群最大节点个数是多少？
25.Redis 集群如何选择数据库？
26.怎么测试 Redis 的连通性？
27.怎么理解 Redis 事务？
28.Redis 事务相关的命令有哪几个？
29.Redis key 的过期时间和永久有效分别怎么设置？
30.Redis 如何做内存优化？
31.Redis 回收进程如何工作的？
32.都有哪些办法可以降低 Redis 的内存使用情况呢？
33.Redis 的内存用完了会发生什么？
34.一个 Redis 实例最多能存放多少的 keys？List、Set、Sorted Set他们最多能存放多少元素？
35.MySQL 里有 2000w 数据，Redis 中只存 20w 的数据，如何保证Redis 中的数据都是热点数据？
36.Redis 最适合的场景是什么？
37.假如 Redis 里面有 1 亿个 key，其中有 10w 个 key 是以某个固定的已知的前缀开头的，如果将它们全部找出来？
38.如果有大量的 key 需要设置同一时间过期，一般需要注意什么？
39.使用过 Redis 做异步队列么，你是怎么用的？
40.使用过 Redis 分布式锁么，它是什么回事？
```

### Redis数据类型

- string类型：Redis中最基本的类型，它是key对应的一个单一值。二进制安全，不必担心由于编码等问题导致二进制数据变化。所以redis的string可以包含任何数据，比如jpg图片或者序列化的对象。Redis中一个字符串值的最大容量是512M。
- list类型：
- Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。它的底层是双向链表，所以它操作时头尾效率高，中间效率低（额外花费查找插入位置的时间）。在Redis中list类型是按照插入顺序排序的字符串链表。和数据结构中的普通链表一样，我们可以在其头部(left)和尾部(right)添加新的元素。在插入时，如果该键并不存在，Redis将为该键创建一个新的链表。与此相反，如果链表中所有的元素均被移除，那么该键也将会被从数据库中删除。List中可以包含的最大元素数量是2^32-1个。list是一个有序可以重复的数据类型。


- set类型：Redis的set是string类型的无序集合。它是基于哈希表实现的。set类型插入数据时会自动去重。最大可以包含2^32-1个元素。
- hash类型：本身就是一个键值对集合。可以当做Java中的Map对待。每一个hash可以存储2^32-1个键值对。
- zset类型：Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的,但分数(score)却可以重复。
- Geospatial：Redis 在 3.2 推出 Geo 类型，该功能可以推算出地理位置信息，两地之间的距离。
- HyperLogLogs：用于大数据量基数统计，速度非常快，占用内存非常小。每个HyperLogLog键只需要花费12KB内存，就可以计算接近 2^64个不同元素的基数。比如计算网站UV（User view，用户访问数量，一个用户一天访问同一个URL地址多次合并为一次）。
- bitmap：直接对string的二进制位进行操作的一组命令

### 常用数据类型应用场景

| 数据类型     | 应用场景                                                |
| ------------ | ------------------------------------------------------- |
| string       | 分布式Session存储 分布式数据库ID 计数器：统计网站访问量 |
| hash         | 存储对象信息（购物车中的商品信息） 存储表的信息         |
| list         | 实现队列、栈操作 汇总日志 粉丝列表 关注的人列表         |
| set          | 签到 打卡 点赞                                          |
| zset         | 排行榜 百度热点搜索                                     |
| geospatial   | 获取地理位置信息 两地之间的距离                         |
| hyperloglogs | 基数统计                                                |
| bitmaps      | 统计用户访问次数                                        |

### Redis常用命令

- redis-server redis.conf 启动服务
- redis-cli：客户端访问
- shutdown：关闭服务

### redis.conf

- bind=127.0.0.1：默认情况bind=127.0.0.1只能接受本机的访问请求；不写的情况下，无限制接受任何ip地址的访问。生产环境肯定要写你应用服务器的地址

- protected-mode：如果开启了protected-mode，那么在没有设定bind ip且没有设密码的情况下，Redis只允许接受本机的相应
- timeout ： 一个空闲的客户端维持多少秒会关闭，0为永不关闭。
- TCP keepalive：对访问客户端的一种心跳检测，每个n秒检测一次。官方推荐设为60秒。
- daemonize ：是否为后台进程。
- pidfile：存放pid文件的位置，每个实例会产生一个不同的pid文件。
- log level：四个级别根据使用阶段来选择，生产环境选择notice 或者warning。
- logfile：日志文件名称。
- syslog：是否将Redis日志输送到linux系统日志服务中。
- syslog-ident：日志的标志。
- syslog-facility ：输出日志的设备。
- database：设定库的数量 默认16。

### 悲观锁(Pessimistic Lock)

- 顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。

### 乐观锁(Optimistic Lock)

- 顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量。Redis就是利用这种check-and-set机制实现事务的。
