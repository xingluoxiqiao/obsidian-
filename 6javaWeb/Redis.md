
Redis（Remote Dictionary Server )，即远程字典服务 !  
是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API；redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步
# NoSQL
![[Pasted image 20230922151933.png]]
# redis特征
键值（ key-value）型,value支持多种不同数据结构，功能丰富
单线程，**每个命令具备原子性**，在网络请求处理方面可以实现多线程
低延迟，速度快（**基于内存**、IO多路复用、良好的编码）
支持数据持久化
支持主从集群、分片集群
支持多语言客户端
# redis通用命令
## help【command】
查看一个命令的具体用法
## KEYS pattern 
查看符合模板的所有key
其中模板指的是redis自身的一些定义，相当于模糊匹配
![[Pasted image 20230922160409.png]]
## DEL 
删除一个指定的key
## EXISTS 
判断key是否存在
## EXPIRE
给一个key设置有效期，有效期到期时该key会被自动删除

## TTL
查看一个key的剩余有效期
# key的层级格式
Redis没有类似MySQL中的Table的概念，s使用key的分层级形式来区分不同类型的key
![[Pasted image 20230922160953.png]]

# redis的数据结构
redis是键值对型数据库，其键一般是字符串，而值的类型多种多样，以下是一些常见的值的类型的介绍
## string
String类型，也就是字符串类型，是Redis中最简单的存储类型。
其value是字符串，不过根据字符串的格式不同，又可以分为3类:
string:普通字符串
int:整数类型，可以做自增、自减操作
float:浮点类型，可以做自增、自减操作
不管是哪种格式，底层都是字节数组形式存储，只不过是编码方式不同。字符串类型的最大空间不能超过512m.
![[Pasted image 20230922160736.png]]
## hash
![[Pasted image 20230922161203.png]]
![[Pasted image 20230922161224.png]]
## list
Redis中的List类型与Java中的LinkedList类似，可以看做是一个双向链表结构。
既可以支持正向检索和也可以支持反向检索。
特征也与LinkedList类似：
有序；元素可以重复；插入和删除快；查询速度一般
![[Pasted image 20230922161424.png]]
可以用list来模拟栈，队列，阻塞队列等数据结构
入口和出口在同一边---->栈
入口和出口在不同边---->队列
入口和出口在不同边，出队时采用BLPOP或BRPOP---->阻塞队列
## set
Redis的Set结构与Java中的HashSet类似，可以看做是一个value为null的HashMap。
因为也是一个hash表，因此具备与HashSet类似的特征：
无序；元素不可重复；查找快；支持交集、并集、差集等功能
![[Pasted image 20230922161828.png]]
## sortedset
Redis的SortedSet是一个可排序的set集合，与Java中的TreeSet有些类似，但底层数据结构却差别很大。
SortedSet中的每一个元素都带有一个score属性，可以基于score属性对元素排序，底层的实现是一个
跳表（SkipList)加 hash表。
SortedSet具备下列特性:
可排序；元素不重复；查询速度快
因为SortedSet的可排序特性，经常被用来实现排行榜这样的功能。
![[Pasted image 20230922164154.png]]
# redis的java客户端
jedis：以Redis命令作为方法名称。学习成本低,简单实用。但是Jedis实例是线程不安全的,多线程环境卞需要基于连接池来使用
lettuce：是基于Netty实现的,支持同步、异步和响应式编程方式，并且是线程家全的。支持Redis的哨兵模式、集群模式和管道模式。
## jedis快速入门
官网：[https://github.com/redis/jedis]
1.引入依赖
```
<dependency>
	<grouprd>redis.clients</grouprd>
	<artifactId>jedis</artifactId>
	<version>3.7.0</version>
</dependency>
```
2.建立连接
![[Pasted image 20230922164616.png]]
3.测试string
![[Pasted image 20230922164635.png]]
4.释放资源
![[Pasted image 20230922164647.png]]
## jedis连接池
jedis本身是线程不安全的，并且频繁的创建和销毁连接会有性能损耗
因此使用jedis连接池代替jedis的直连方式
![[Pasted image 20230922164951.png]]
## SpringDataRedis
springdata是spring中数据操作的模块，包含对各种数据的集成，其中对redis的集成模块就是SpringDataRedis，官网:[https:/lsprina.io/proiects/spring-data-redis]
它提供了如下功能和便利：
1.提供了对不同Redis客户端的整合（Lettuce和Jedis)
2.提供了RedisTemplate统一API来操作Redis
3.支持Redis的发布订阅模型
4.支持Redis哨兵和Redis集群
5.支持基于Lettuce的响应式编程
6.支持基于JDK、JSON、字符串、Spring对象的数据序列化及反序列化
7.支持基于Redis的JDKCollection实现

### 快速入门

1.引入依赖
```
<!--Redis依赖-->
<dependency>
	<groupid>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!--连接池依赖-->
<dependency>
	<groupid>org.apache.commons</groupid>
	<artifactId>commons-pool2</artifactId>
</dependency>
```
2.配置文件
```
spring:
	redis:
		host: 192.168.150.101
		port: 6379
		password: 123321
		lettuce:
			pool:
				max-active: 8#最大连接
				max-idle: 8#晟大空闲连接
				min-idle: 0#最小空闲连接
				max-wait: 100 #连接等待时间
```
3.注入RedisTemplate
```
@Autowired
private RedisTemplate redisTemplate
```
4.编写测试
![[Pasted image 20230922165941.png]]
如果要操作其它数据类型，可以参照下表以及redis中各数据类型的命令（jedis中的方法名与命令名相同），用下表的对应类调用对应方法
![[Pasted image 20230922165439.png]]
### 序列化方式
RedisTemplate可以接受任意Object作为值写入redis，但是写入前会把Object序列化为字节形式，默认是采用JDK序列化，这样做使得值的可读性差，并且内存占用较大，为了解决这个问题，有两种方式
**自定义RedisTemplate的序列化方式**
![[Pasted image 20230922170235.png]]
以上是采用jackson序列化的示例，注意需要引入jackson依赖
```
<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-databind</artifactId>
</dependency>
```
尽管它能改善可读性差的问题，但是为了在反序列化时知道对象的类型，JSON序列化器会将类的class类型写入json结果中，存入Redis，会带来额外的内存开销。
**使用Spring提供的一个StringRedisTemplate类**
为了节省内存空间，我们并不会使用JSON序列化器来处理value，而是统一使用String序列化器，它要求只能存储String类型的key和value。当需要存储Java对象时，我们需要**手动完成**对象的序列化和反序列化。
StringRedisTemplate的key和value的序列化方式默认就是String方式。省去了我们自定义RedisTemplate的过程
![[Pasted image 20230922170840.png]]
![[Pasted image 20230922171119.png]]
# 事务和乐观锁
## 事务
1.原子性（atomicity）。一个事务是一个不可分割的工作单位，事务中包括的操作要么都做，要么都不做。
2.一致性（consistency）。事务必须是使数据库从一个一致性状态变到另一个一致性状态。一致性与原子性是密切相关的。
3.隔离性（isolation）。一个事务的执行不能被其他事务干扰。即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。
4.持久性（durability）。持久性也称永久性（permanence），指一个事务一旦提交，它对数据库中数据的改变就应该是永久性的。接下来的其他操作或故障不应该对其有任何影响。
在Redis事务没有隔离级别的概念！
在Redis单条命令是保证原子性的，但是事务不保证原子性！
## 乐观锁
1.当程序中可能出现并发的情况时，就需要保证在并发情况下数据的准确性，以此确保当前用户和其他用户一起操作时，所得到的结果和他单独操作时的结果是一样的。
2.没有做好并发控制，就可能导致脏读、幻读和不可重复读等问题。
在Redis是可以实现乐观锁的！
## 事务的实现
一、Redis如何实现事务？
1.正常执行事务
![[Pasted image 20230930154014.png]]
2.放弃事务
![[Pasted image 20230930154026.png]]
3.编译时异常，代码有问题，或者命令有问题，所有的命令都不会被执行
![[Pasted image 20230930154041.png]]
4.运行时异常，除了语法错误不会被执行且抛出异常后，其他的正确命令可以正常执行
![[Pasted image 20230930154059.png]]
5.总结：由以上可以得出结论，Redis是支持单条命令事务的，但是事务并不能保证原子性！
## 乐观锁的实现
1.watch（监视）
![[Pasted image 20230930154234.png]]
2.多线程测试watch
![[Pasted image 20230930154302.png]]
![[Pasted image 20230930154313.png]]
3.总结：乐观锁和悲观锁的区别：
悲观锁： 什么时候都会出问题，所以一直监视着，没有执行当前步骤完成前，不让任何线程执行，十分浪费性能！一般不使用！
乐观锁： 只有更新数据的时候去判断一下，在此期间是否有人修改过被监视的这个数据，没有的话正常执行事务，反之执行失败！
# 持久化
Redis 是**内存数据库**，如果不将内存中的数据库状态保存到磁盘，那么一旦服务器进程退出，服务器中的数据库状态也会消失。所以 Redis 提供了**持久化功能** !
## RDB（Redis DataBase）
RDB持久化是Redis的一种快照持久化方式，它可以将内存中的数据周期性地保存到磁盘上的一个二进制文件中。这个文件包含了某个时间点上的所有数据，以及服务器的状态信息。RDB持久化的主要特点和步骤如下：
1. **快照生成**：Redis会定期生成一个快照文件，保存当前数据和服务器状态。你可以通过配置Redis的`save`指令来指定生成快照的条件，比如多少秒内至少有多少个写操作。
2. **生成快照文件**：生成快照文件时，Redis会 fork 一个子进程来执行实际的快照生成操作，而父进程则继续响应客户端请求。这样可以确保持久化操作不会阻塞Redis的正常服务。
3. **保存到磁盘**：生成的快照文件会被保存到磁盘上的一个文件中。
4. **加载快照**：当Redis服务器启动时，它会检查是否存在RDB快照文件，如果存在，就会加载该文件并将数据还原到内存中。

Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的。
这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。我们默认的就是RDB，一般情况下不需要修改这个配置！在生产环境我们会将这个文件进行备份！
### 快照生成机制（生成dump.rdb文件）
1.在redis的配置文件中修改对应区域可以指定快照生成的条件，满足条件生成快照
![[Pasted image 20230930155617.png]]
![[Pasted image 20230930155652.png]]
2.执行flushall命令，也会触发rdb规则
3.退出Redis，也会触发rdb规则
4.手动执行save命令生成快照文件
### 恢复快照文件
一般redis每次重启时会自动加载快照文件，实现持久化
1、只需将备份的rdb文件放在我们的redis启动目录即可，Redis启动的时候会自动检查dump.rdb文件并恢复其中的数据！
2、查找文件位置的命令：
### 优缺点
优点：
1、适合大规模的数据恢复！
2、对数据的完整性要求不高！
3、性能高：生成快照时使用了子进程，不会影响正常的读写操作
4、生成的快照文件紧凑，适用于备份和恢复
缺点：
1、需要一定的时间间隔进程操作！如果redis意外宕机了，这个最后一次修改数据就没有的了！
2、fork进程的时候，会占用一定的内容空间！
3、不适用于实时数据备份：生成快照的频率较低，不适用于要求实时数据备份的场景。
## AOF（Append Only File）
Redis默认使用的是RDB模式，所以需要手动开启AOF模式，在配置文件中将下图中的no改为yes
![[Pasted image 20230930160809.png]]
开启AOF模式后，redis会自动保存从这次启动redis服务器以来操作的命令，并记录到appendonly.aof文件中，从Redis 2.0版本开始，AOF持久化就已经是默认启用的持久化方式。
### appendonly.aof错误修复
由于appendonly.aof文件是可读写的，因此有可能产生错误或遭到破坏，可以通过以下方法修复
1. **备份原始AOF文件**：首先，确保在尝试恢复之前备份原始的`appendonly.aof`文件。这可以帮助你在恢复过程中避免进一步的数据损坏。
2. **检查文件完整性**：使用文本编辑器打开`appendonly.aof`文件，并检查文件是否完整和有效。有时，AOF文件可能会因某种原因损坏，导致无法正常解析其中的命令。如果文件完全无法打开或损坏严重，可能需要查看备份或考虑其他数据恢复方式。
3. **手动编辑文件**：如果文件中只有一小部分数据受损，可以尝试手动编辑文件，修复损坏的部分。这可能需要一些Redis命令和AOF文件格式的了解。务必小心操作，以免进一步破坏文件。
4. **使用Redis-check-aof工具**：Redis提供了一个名为`redis-check-aof`的工具，可以用于检查AOF文件的有效性并尝试修复其中的问题。可以通过以下命令来使用它：`redis-check-aof --fix <AOF文件路径>`
该命令将尝试修复AOF文件中的问题，并在修复完成后生成一个修复后的文件（通常带有`.fixed`扩展名）。可以将修复后的文件重命名为`appendonly.aof`并替换原始文件。
注意虽然错误的内容少了，但是正确的也有一定的丢失！所以这个修复无法做到百分百修复！
5. **重新加载AOF文件**：如果成功修复了AOF文件或者恢复了损坏的部分，可以重新启动Redis服务器以加载AOF文件中的命令并还原数据。
### 设置appendonly.aof文件大小
aof默认的就是文件的无限追加，文件会越来越大！在配置文件中可以设置文件的大小！
![[Pasted image 20230930161737.png]]
```
auto-aof-rewrite-percentage 100 #写入百分比 
auto-aof-rewrite-min-size 64mb #写入的文件最大值是多少，一般在实际工作中我们会将其设置为5gb左右！
```
### 优缺点
优点：
1. **可读性和透明性**：AOF文件是一个可读性的文本文件，它记录了每个写操作的命令。这使得AOF文件易于查看和理解，有助于调试和分析。
2. **实时备份**：AOF模式以追加的方式记录每个写操作，这意味着数据变化会立即被记录到AOF文件中。这使得AOF模式适用于实时数据备份需求。每一次修改都同步，文件的完整性会更加好
3. **可靠性**：AOF文件采用了追加写入方式，相对于RDB持久化，更不容易损坏。即使在写入过程中发生意外宕机，已经写入的数据不会丢失。每秒同步一次，最多会丢失一秒的数据
4. **重写机制**：Redis提供了AOF文件的重写机制，允许定期对AOF文件进行重新压缩和优化。这可以控制AOF文件的大小，避免无限增长。
5. **数据恢复**：AOF文件记录了写操作的历史，因此可以用于恢复数据。在Redis服务器启动时，AOF文件中的命令将会重新执行，还原数据。
缺点：
1. **文件大小**：AOF文件通常会随着时间的推移逐渐增大，特别是在高写入负载下。较大的AOF文件可能占用大量磁盘空间，因此需要定期进行AOF文件的重写和优化。
2. **写入性能**：相对于RDB持久化，AOF持久化在高写入负载下可能会稍微降低性能，因为每个写操作都要追加到AOF文件中。
3. **文件恢复时间**：如果AOF文件过大，Redis服务器在启动时加载AOF文件的时间可能会较长，因为需要重新执行大量写操作。
4. **数据丢失风险**：虽然AOF文件相对可靠，但在某些极端情况下，可能会出现数据丢失。例如，如果AOF文件在写入期间发生了损坏，那么写入的数据可能会丢失。
## 两种方式对比和总结
1、RDB 持久化方式能够在指定的时间间隔内对数据进行快照存储
2、AOF 持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以Redis 协议追加保存每次写的操作到文件末尾，Redis还能对AOF文件进行后台重写，使得AOF文件的体积不至于过大。
3、只做缓存，如果只希望数据在服务器运行的时候存在，也可以不使用任何持久化
4、同时开启两种持久化方式时，当redis重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。
RDB 的数据不实时，同时使用两者时服务器重启也只会找AOF文件，但建议不要只使用AOF，因为RDB更适合用于备份数据库（AOF在不断变化不好备份），快速重启，而且不会有AOF可能潜在的Bug，留着作为一个万一的手段。
5.一般情况下无脑两种一起用，此外
**使用AOF持久化的情况**：
1. **实时备份需求**：如果你需要实时备份数据以确保数据不会丢失，AOF持久化是更好的选择。AOF以追加方式记录每个写操作，确保数据变更会立即记录到AOF文件中。
2. **可读性和调试需求**：AOF文件是可读性的文本文件，易于查看和理解其中的命令。这对于调试和分析非常有用。
3. **数据恢复要求**：AOF文件记录了写操作的历史，因此可以用于数据恢复。在Redis服务器启动时，AOF文件中的命令将会重新执行，还原数据。
4. **数据一致性要求高**：AOF模式相对可靠，即使在写入过程中发生宕机，已经写入的数据不会丢失，因此适用于要求数据一致性高的场景。
**使用RDB持久化的情况**：
1. **周期性备份需求**：如果你只需要定期备份数据，而不需要实时备份，RDB持久化是一种有效的选择。RDB生成全量快照，适用于周期性的备份操作。
2. **磁盘空间有限**：RDB文件通常比较小，适用于磁盘空间有限的情况。如果你的磁盘空间有限，可以考虑使用RDB持久化。
3. **启动速度要求**：在Redis服务器启动时，加载RDB文件比加载大型AOF文件更快。如果需要快速启动，可以选择RDB持久化。
4. **性能优化**：在某些高性能场景下，RDB持久化可能会比AOF持久化更适合，因为RDB生成快照的性能开销较小。
## 性能建议
1.因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留 save 900 1 这条规则。
2.如果Enable AOF ，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了，代价一是带来了持续的IO，二是AOF rewrite 的最后将 rewrite 过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率
AOF重写的基础大小默认值64M太小了，可以设到5G以上，默认超过原大小100%大小重写可以改到适当的数值。
3.如果不Enable AOF ，仅靠 Master-Slave Repllcation 实现高可用性也可以，能省掉一大笔IO，也减少了rewrite时带来的系统波动。代价是如果Master/Slave 同时倒掉，会丢失十几分钟的数据，启动脚本也要比较两个 Master/Slave 中的 RDB文件，载入较新的那个，微博就是这种架构。
# 发布订阅
Redis发布订阅（pub/sub）是一种消息通信模式：发送者（pub）发送消息，订阅者（sub）接受消息。
![[Pasted image 20230930174823.png]]
## 实现
订阅端
![[Pasted image 20230930174926.png]]
发送端
![[Pasted image 20230930174936.png]]
## 常用命令
**发布消息到指定频道**：
**PUBLISH**：将消息发布到指定的频道。
    `PUBLISH channel message`
    - `channel`：指定要发布消息的频道名称。
    - `message`：要发布的消息内容。
**订阅频道**：
1. **SUBSCRIBE**：订阅一个或多个频道。   
    `SUBSCRIBE channel [channel ...]`
    - `channel`：一个或多个频道名称，可以同时订阅多个频道。
2. **PSUBSCRIBE**：通过正则表达式订阅匹配的频道。
    `PSUBSCRIBE pattern [pattern ...]`
    - `pattern`：一个或多个正则表达式，用于匹配多个频道名称。
**取消订阅频道**：
1. **UNSUBSCRIBE**：取消订阅一个或多个频道。
    `UNSUBSCRIBE [channel [channel ...]]`
    - `channel`：要取消订阅的频道名称，如果未提供任何频道名称，则取消所有频道的订阅。
2. **PUNSUBSCRIBE**：通过正则表达式取消订阅匹配的频道。
    `PUNSUBSCRIBE [pattern [pattern ...]]`
    - `pattern`：要取消订阅的正则表达式，如果未提供任何正则表达式，则取消所有匹配的频道的订阅。
**查看订阅频道**：
1. **SUBSCRIBE** 和 **PSUBSCRIBE** 命令会返回关于订阅状态的信息。你可以使用以下命令来查看当前订阅的频道和模式：
    - `PUBSUB CHANNELS`：列出当前活动的频道。
    - `PUBSUB NUMSUB channel [channel ...]`：获取指定频道的订阅者数量。
    - `PUBSUB NUMPAT`：获取匹配的模式数量。
# 主从复制
主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master/leader)，后者称为从节点(slave/follower)；数据的复制是单向的，只能由主节点到从节点。Master以写为主，Slave 以读为主。
主要作用：
1.数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
2.故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
3.负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。
4.高可用（集群）基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。
## 环境配置（单机集群）
1.基本查看命令info replication
![[Pasted image 20230930175500.png]]
2.例开启三台服务
![[Pasted image 20230930175635.png]]
3.全部启动并查看
![[Pasted image 20230930175658.png]]
## 单机测试（一主二从）
1.任命一台服务器为主节点，其它服务器为从节点slaveof  IP port
![[Pasted image 20230930175810.png]]
2.从主节点处查看信息info replication
![[Pasted image 20230930175847.png]]
3.以上的配置是一次性的，如果断电、宕机等，就要重新任命
可以通过修改配置文件来实现永久配置
![[Pasted image 20230930180117.png]]
4.测试读写操作
主机写，从机可读
主机死，从机可读
主机复活，从机自动寻找主机（配置文件配置时）
从机死，不可重连（命令配置时）
从机只能读，不能写
## 原理
Slave 启动成功连接到 master 后会发送一个sync同步命令
Master 接到命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到slave，并完成一次完全同步。

全量复制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。
增量复制： Master 继续将新的所有收集到的修改命令依次传给slave，完成同步但是只要是重新连接master，一次完全同步（全量复制）将被自动执行！ 我们的数据一定可以在从机中看到
## 从机的从机
层层链路
![[Pasted image 20230930180522.png]]
从机可以有自己的从机（主从机的概念是相对的）
## 主机转移（谋朝篡位）
使用`SLAVEOF no one`让自己变成主机！其他的节点就可以手动连接到最新的这个主节点（手动） 
如果主机复活，重新成为从机
## 小结
一般来说，要将Redis运用于工程项目中，只使用一台Redis是万万不能的（宕机），原因如下：
1、从结构上，单个Redis服务器会发生单点故障，并且一台服务器需要处理所有的请求负载，压力较大；
2、从容量上，单个Redis服务器内存容量有限，就算一Redis服务器内存容量为256G，也不能将所有内存用作Redis存储内存，一般来说，单台Redis最大使用内存不应该20G。
主从复制，读写分离！ 80% 的情况下都是在进行读操作！减缓服务器的压力！架构中经常使用！ 一主二从！
只要在公司中，主从复制就是必须要使用的，因为在真实的项目中不可能单机使用Redis！
# 哨兵模式
是对主从复制的补充
主机断开后，我们得手动设置另一个从机变成主机！这是不智能的！在实际工作中，我们都是用哨兵模式来自动切换主机。Redis从2.8开始正式提供了Sentinel（哨兵） 架构，后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库。  
哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的 **进程** ，作为进程，它会独立运行。其原理是哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。
## 配置哨兵
1.添加哨兵配置文件sentinel.conf
内容如下：
```
# sentinel monitor 被监控的名称 host port 1 （代表自动投票选举大哥！）
sentinel monitor myredis 127.0.0.1 6379 1
```
2.启动哨兵
```
redis-sentinel sentinel配置文件路径   #和启动Redis一致
```
3.准备测试环境（一主二从三台服务器）
4.测试主机宕机后自动选取大哥，如果主机此时回来了，只能归并到新的主机下，当做从机，这就是哨兵模式的规则！等待哨兵的默认配置时间时是30 秒！
## 优缺点
优点
1.哨兵集群基于主从复制模式 ，所有的主从配置优点，它全有
2.主从可以切换，故障可以转移 ，系统的 可用性 就会更好
3.哨兵模式就是主从模式的升级，手动到自动，更加健壮
缺点
1.Redis 不好在线扩容 的，集群容量一旦到达上限，在线扩容就十分麻烦！
2.实现哨兵模式的配置其实是很 麻烦 的，里面有很多选择！
注意
以上所有的配置因为条件所限都是基于单机集群的前提下！正式集群下的多哨兵模式如下图：
![[Pasted image 20230930181510.png]]
## 哨兵的配置文件解析
```
# Example sentinel.conf 

# 哨兵sentinel实例运行的端口 默认26379 
port 26379 

# 哨兵sentinel的工作目录 
dir /tmp 

# 哨兵sentinel监控的redis主节点的 ip port 
# master-name 可以自己命名的主节点名字 只能由字母A-z、数字0-9 、这三个字符".-_"组成。 
# quorum 配置多少个sentinel哨兵统一认为master主节点失联 那么这时客观上认为主节点失联了 
# sentinel monitor <master-name> <ip> <redis-port> <quorum> sentinel monitor mymaster 127.0.0.1 6379 2 

# 当在Redis实例中开启了requirepass foobared 授权密码 这样所有连接Redis实例的客户端都要提供 密码
# 设置哨兵sentinel 连接主从的密码 注意必须为主从设置一样的验证密码 
# sentinel auth-pass <master-name> <password> 
sentinel auth-pass mymaster MySUPER--secret-0123passw0rd 

# 指定多少毫秒之后 主节点没有应答哨兵sentinel 此时 哨兵主观上认为主节点下线 默认30秒 
# sentinel down-after-milliseconds <master-name> <milliseconds> 
sentinel down-after-milliseconds mymaster 30000 

# 这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行 同步
#这个数字越小，完成failover所需的时间就越长，
# 但是如果这个数字越大，就意味着越 多的slave因为replication而不可用。 
#可以通过将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态。 
# sentinel parallel-syncs <master-name> <numslaves> 
sentinel parallel-syncs mymaster 1 


# 故障转移的超时时间 failover-timeout 可以用在以下这些方面： 
#1. 同一个sentinel对同一个master两次failover之间的间隔时间。 
#2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那 里同步数据时。 
#3.当想要取消一个正在进行的failover所需要的时间。 
#4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时， slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了 
# 默认三分钟 # sentinel failover-timeout <master-name> 
sentinel failover-timeout mymaster 180000 


# SCRIPTS EXECUTION #配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知 相关人员。 
#对于脚本的运行结果有以下规则： 
#若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10 #若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。 
#如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。 
#一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。 
#通知型脚本:当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等）， 将会去调用这个脚本，这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信 息。调用该脚本时，将传给脚本两个参数，一个是事件的类型，一个是事件的描述。如果sentinel.conf配 置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentinel无 法正常启动成功。 
#通知脚本 
# shell编程 
# sentinel notification-script <master-name> <script-path> 
sentinel notification-script mymaster /var/redis/notify.sh 


# 客户端重新配置主节点参数脚本 
# 当一个master由于failover而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已 经发生改变的信息。 
# 以下参数将会在调用脚本时传给脚本: 
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port> 
# 目前<state>总是“failover”, 
# <role>是“leader”或者“observer”中的一个。 
# 参数 from-ip, from-port, to-ip, to-port是用来和旧的master和新的master(即旧的slave)通 信的
# 这个脚本应该是通用的，能被多次调用，不是针对性的。 
# sentinel client-reconfig-script <master-name> <script-path> 
sentinel client-reconfig-script mymaster /var/redis/reconfig.sh 
# 一般都是由运维来配置！
```
# 缓存穿透
用户需要查询一个数据，但是redis中没有（比如说mysql中id=-1的数），直接去请求MySQL，当很多用户同时请求并且都没有命中！于是都去请求了持久层的数据库，那么这样会给持久层数据库带来非常大的压力。一般出现这样的情况都不是正常用户，基本上都是恶意用户！
**缓存穿透前提是：Redis和MySQL中都没有，然后不停的直接请求MySQL。**
## 解决方案
### 布隆过滤器
![[Pasted image 20230930181912.png]]
布隆过滤器是一种数据结构，对所有可能查询的参数以hash形式存储，在控制层先进行校验，不符合则  
丢弃，从而避免了对底层存储系统的查询压力；
### 缓冲空对象
![[Pasted image 20230930181949.png]]
当存储层查不到，即使是空值，我们也将其存储起来并且在Redis中设置一个过期时间，之后再访问这个数据将会从Redis中访问，保护了持久层的数据库！
但是如果空值能够被缓存起来，这就意味着缓存需要更多的空间存储更多的键，因为这当中可能会有很多  
的空值的键；  即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这对于需要保持一致性的业务会有影响。
# 缓存击穿
是指一个非常热点的key，在不停的扛着大并发，当这个key失效时，一瞬间大量的请求冲到持久层的数据库中，就像在一堵墙上某个点凿开了一个洞！
## 解决方案
1.设置热点key永不过期
从缓存层面来看，没有设置过期时间，所以不会出现热点 key 过期后产生的问题。这样做其实并不合理
2.加互斥锁
在查询持久层数据库时，保证了只有一个线程能够进行持久层数据查询，其他的线程让它睡眠几百毫秒，等待第一个线程查询完会回写到Redis缓存当中，剩下的线程可以正常查询Redis缓存，就不存在大量请求去冲击持久层数据库了！
![[Pasted image 20230930182240.png]]
# 缓存雪崩
在某一个时间段，缓存的key大量集中同时过期了，所有的请求全部冲到持久层数据库上，导致持久层数据库挂掉！  
范例：双十一零点抢购，这波商品比较集中的放在缓存，设置了失效时间为1个小时，那么到了零点，这批缓存全部失效了，而大量的请求过来时，全部冲过了缓存，冲到了持久层数据库！
## 解决方案
### Redis高可用
搭建Redis集群，既然redis有可能挂掉，那么多增设几台redis，这样一台挂掉之后其他的还可以继续工作，其实就是搭建的集群。（异地多活！）
### 限流降级
在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。
### 数据预热
数据加热的含义就是在正式部署之前，先把可能的数据先预先访问一遍，这样部分可能大量访问的数据就会加载到缓存中。在即将发生大并发访问前手动触发加载缓存不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀 。
# 配置文件详解
redis.conf 中的各个参数意义：
1. **daemonize no**
默认情况下，redis 不是在后台运行的，如果需要在后台运行，把该项的值更改为 yes。
2. pidfile /var/run/redis.pid
当 Redis 在后台运行的时候，Redis 默认会把 pid 文件放在/var/run/redis.pid，你可以配置到其他地址。当运行多个 redis 服务时，需要指定不同的 pid 文件和端口。
3. **port**
监听端口，默认为 6379。
4. **\#bind 127.0.0.1**
指定 Redis只接收来自于该 IP 地址的请求，如果不进行设置，那么将处理所有请求，在生产环境中为了安全最好设置该项。默认注释掉，不开启。
5. **timeout 0**
设置客户端连接时的超时时间，单位为秒。当客户端在这段时间内没有发出任何指令，那么关闭该连接。
6. tcp-keepalive 0
指定 TCP 连接是否为长连接,"侦探"信号有 server 端维护。默认为 0.表示禁用。
7. **loglevel notice**
log 等级分为 4 级，debug,verbose, notice, 和 warning。生产环境下一般开启 notice。
8. logfile stdout
配置 log 文件地址，默认使用标准输出，即打印在命令行终端的窗口上，修改为日志文件目录。
9. **databases 16**
设置数据库的个数，可以使用 SELECT 命令来切换数据库。默认使用的数据库是 0 号库。默认 16 个库。
10.
**save 900 1
save 300 10
save 60 10000**
保存数据快照的频率，即将数据持久化到 dump.rdb 文件中的频度。用来描述"在多少秒期间至少多少个变更操作"触发 snapshot 数据保存动作。
默认设置，意思是：
if(在 60 秒之内有 10000 个 keys 发生变化时){
进行镜像备份
}else if(在 300 秒之内有 10 个 keys 发生了变化){
进行镜像备份
}else if(在 900 秒之内有 1 个 keys 发生了变化){
进行镜像备份
}
11. stop-writes-on-bgsave-error yes
当持久化出现错误时，是否依然继续进行工作，是否终止所有的客户端 write 请求。默认设置"yes"表示终止，一旦 snapshot 数据保存故障，那么此 server 为只读服务。如果为"no"，那么此次 snapshot 将失败，但下一次 snapshot 不会受到影响，不过如果出现故障,数据只能恢复到"最近一个成功点"。
12. rdbcompression yes
在进行数据镜像备份时，是否启用 rdb 文件压缩手段，默认为 yes。压缩可能需要额外的cpu 开支，不过这能够有效的减小 rdb 文件的大，有利于存储/备份/传输/数据恢复。
13. rdbchecksum yes
读取和写入时候，会损失 10%性能是否进行校验和，是否对 rdb 文件使用 CRC64 校验和,默认为"yes"，那么每个 rdb 文件内容的末尾都会追加 CRC 校验和，利于第三方校验工具检测文件完整性。
14. **dbfilename dump.rdb**
镜像备份文件的文件名，默认为 dump.rdb。
15. dir ./
数据库镜像备份的文件 rdb/AOF 文件放置的路径。这里的路径跟文件名要分开配置是因为Redis 在进行备份时，先会将当前数据库的状态写入到一个临时文件中，等备份完成时，再把该临时文件替换为上面所指定的文件，而这里的临时文件和上面所配置的备份文件都会放在这个指定的路径当中。
16. **# slaveof masterip masterport
\# slaveof \<masterip> \<masterport>**
设置该数据库为其他数据库的从数据库，并为其指定 master 信息。
17. masterauth
当主数据库连接需要密码验证时，在这里指定。
18. slave-serve-stale-data yes
当主master服务器挂机或主从复制在进行时，是否依然可以允许客户访问可能过期的数据。在"yes"情况下,slave 继续向客户端提供只读服务,有可能此时的数据已经过期；在"no"情况下，任何向此 server 发送的数据请求服务(包括客户端和此 server 的 slave)都将被告知“error”。
19. **slave-read-only yes**
slave 是否为"只读"，强烈建议为"yes"。
20. # repl-ping-slave-period 10
slave 向指定的 master 发送 ping 消息的时间间隔(秒)，默认为 10。
21. # repl-timeout 60
slave 与 master 通讯中,最大空闲时间,默认 60 秒.超时将导致连接关闭。
22. repl-disable-tcp-nodelay no
slave 与 master 的连接,是否禁用 TCP nodelay 选项。"yes"表示禁用,那么 socket 通讯中数据将会以 packet 方式发送(packet 大小受到 socket buffer 限制)。可以提高 socket通讯的效率(tcp交互次数),但是小数据将会被buffer,不会被立即发送,对于接受者可能存在延迟。"no"表示开启 tcp nodelay 选项,任何数据都会被立即发送,及时性较好,但是效率较低，建议设为 no。
23. slave-priority 100
适用 Sentinel 模块(unstable,M-S 集群管理和监控),需要额外的配置文件支持。slave 的权重值,默认100.当master失效后,Sentinel将会从slave列表中找到权重值最低(>0)的slave,并提升为 master。如果权重值为 0,表示此 slave 为"观察者",不参与 master 选举。
24. # requirepass foobared
设置客户端连接后进行任何其他指定前需要使用的密码。警告：因为 redis 速度相当快，所以在一台比较好的服务器下，一个外部的用户可以在一秒钟进行 150K 次的密码尝试，这意味着你需要指定非常非常强大的密码来防止暴力破解。
25. # rename-command CONFIG 3ed984507a5dcd722aeade310065ce5d (方式:MD5(‘CONFIG^!’))
重命名指令,对于一些与"server"控制有关的指令,可能不希望远程客户端(非管理员用户)链接随意使用,那么就可以把这些指令重命名为"难以阅读"的其他字符串。
26. **# maxclients 10000**
限制同时连接的客户数量。当连接数超过这个值时，redis 将不再接收其他连接请求，客户端尝试连接时将收到 error 信息。默认为 10000，要考虑系统文件描述符限制，不宜过大，浪费文件描述符，具体多少根据具体情况而定。
27. # maxmemory bytes
\# maxmemory \<bytes>
redis-cache 所能使用的最大内存(bytes),默认为 0,表示"无限制",最终由 OS 物理内存大小决定(如果物理内存不足,有可能会使用 swap)。此值尽量不要超过机器的物理内存尺寸,从性能和实施的角度考虑,可以为物理内存 3/4。此配置需要和"maxmemory-policy"配合使用,当 redis 中内存数据达到 maxmemory 时,触发"清除策略"。在"内存不足"时,任何 write 操作(比如 set,lpush 等)都会触发"清除策略"的执行。在实际环境中,建议 redis 的所有物理机器的硬件配置保持一致(内存一致),同时确保 master/slave 中"maxmemory""policy"配置一致。
当内存满了的时候，如果还接收到 set 命令，redis 将先尝试剔除设置过 expire 信息的key，而不管该 key 的过期时间还没有到达。在删除时，将按照过期时间进行删除，最早将要被过期的 key 将最先被删除。如果带有 expire 信息的 key 都删光了，内存还不够用，那么将返回错误。这样，redis 将不再接收写请求，只接收 get 请求。maxmemory 的设置比较适合于把 redis 当作于类似 memcached 的缓存来使用。
28. # maxmemory-policy volatile-lru
内存不足"时,数据清除策略,默认为"volatile-lru"。
volatile-lru ->对"过期集合"中的数据采取 LRU(近期最少使用)算法.如果对 key 使用"expire"指令指定了过期时间,那么此 key 将会被添加到"过期集合"中。将已经过期/LRU 的数据优先移除.如果"过期集合"中全部移除仍不能满足内存需求,将 OOM。
allkeys-lru ->对所有的数据,采用 LRU 算法。
volatile-random ->对"过期集合"中的数据采取"随即选取"算法,并移除选中的 K-V,直到"内存足够"为止. 如果如果"过期集合"中全部移除全部移除仍不能满足,将 OOM。
allkeys-random ->对所有的数据,采取"随机选取"算法,并移除选中的 K-V,直到"内存足够"为止。
volatile-ttl ->对"过期集合"中的数据采取 TTL 算法(最小存活时间),移除即将过期的数据。
noeviction ->不做任何干扰操作,直接返回 OOM 异常。
另外，如果数据的过期不会对"应用系统"带来异常,且系统中 write 操作比较密集,建议采取"allkeys-lru"。
29. # maxmemory-samples 3
默认值 3，上面 LRU 和最小 TTL 策略并非严谨的策略，而是大约估算的方式，因此可以选择取样值以便检查。
29. **appendonly no**
默认情况下，redis 会在后台异步的把数据库镜像备份到磁盘，但是该备份是非常耗时的，而且备份也不能很频繁。所以 redis 提供了另外一种更加高效的数据库备份及灾难恢复方式。开启 append only 模式之后，redis 会把所接收到的每一次写操作请求都追加appendonly.aof 文件中，当 redis 重新启动时，会从该文件恢复出之前的状态。但是这样会造成 appendonly.aof 文件过大，所以 redis 还支持了 BGREWRITEAOF 指令，对appendonly.aof 进行重新整理。如果不经常进行数据迁移操作，推荐生产环境下的做法为关闭镜像，开启 appendonly.aof，同时可以选择在访问较少的时间每天对 appendonly.aof进行重写一次。
另外，对 master 机器,主要负责写，建议使用 AOF,对于 slave,主要负责读，挑选出 1-2 台开启 AOF，其余的建议关闭。
30. **# appendfilename appendonly.aof**
aof 文件名字，默认为 appendonly.aof。
31. appendfsync aof 同步频率
\# appendfsync always
\# appendfsync everysec
\# appendfsync no
设置对 appendonly.aof 文件进行同步的频率。always 表示每次有写操作都进行同步，everysec 表示对写操作进行累积，每秒同步一次。no 不主动 fsync，由 OS 自己来完成。这个需要根据实际业务场景配置。
32. **no-appendfsync-on-rewrite no**
在 aof rewrite 期间,是否对 aof 新记录的 append 暂缓使用文件同步策略,主要考虑磁盘 IO开支和请求阻塞时间。默认为 no,表示"不暂缓",新的 aof 记录仍然会被立即同步。
33. auto-aof-rewrite-percentage 100
当 Aof log 增长超过指定比例时，重写 log file， 设置为 0 表示不自动重写 Aof 日志，重写是为了使 aof 体积保持最小，而确保保存最完整的数据。
34. auto-aof-rewrite-min-size 64mb
触发 aof rewrite 的最小文件尺寸。
35. lua-time-limit 5000
lua 脚本运行的最大时间。
36. slowlog-log-slower-than 10000
"慢操作日志"记录,单位:微秒(百万分之一秒,1000 * 1000),如果操作时间超过此值,将会把command 信息"记录"起来.(内存,非文件)。其中"操作时间"不包括网络 IO 开支,只包括请求达到 server 后进行"内存实施"的时间."0"表示记录全部操作。
37. slowlog-max-len 128
"慢操作日志"保留的最大条数,"记录"将会被队列化,如果超过了此长度,旧记录将会被移除。可以通过"SLOWLOG args"查看慢记录的信息(SLOWLOG get10,SLOWLOG reset)。
38. hash-max-ziplist-entries 512
hash 类型的数据结构在编码上可以使用 ziplist 和 hashtable。ziplist 的特点就是文件存储(以及内存存储)所需的空间较小,在内容较小时,性能和 hashtable 几乎一样.因此 redis 对hash 类型默认采取 ziplist。如果 hash 中条目的条目个数或者 value 长度达到阀值,将会被重构为 hashtable。
这个参数指的是 ziplist 中允许存储的最大条目个数，，默认为 512，建议为 128。
hash-max-ziplist-value 64
ziplist 中允许条目 value 值最大字节数，默认为 64，建议为 1024。
39.
list-max-ziplist-entries 512
list-max-ziplist-value 64
对于 list 类型,将会采取 ziplist,linkedlist 两种编码类型。解释同上。
40. set-max-intset-entries 512
intset 中允许保存的最大条目个数,如果达到阀值,intset 将会被重构为 hashtable。
41.
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
zset 为有序集合,有 2 中编码类型:ziplist,skiplist。因为"排序"将会消耗额外的性能,当 zset中数据较多时,将会被重构为 skiplist。
42. activerehashing yes
是否开启顶层数据结构的 rehash 功能,如果内存允许,请开启。rehash 能够很大程度上提高K-V 存取的效率。
43.
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
客户端 buffer 控制。在客户端与 server 进行的交互中,每个连接都会与一个 buffer 关联,此buffer 用来队列化等待被 client 接受的响应信息。如果 client 不能及时的消费响应信息,那么 buffer 将会被不断积压而给 server 带来内存压力.如果 buffer 中积压的数据达到阀值,将
会导致连接被关闭,buffer 被移除。
buffer 控制类型包括:normal -> 普通连接；slave ->与 slave 之间的连接；pubsub ->pub/sub 类型连接，此类型的连接，往往会产生此种问题;因为 pub 端会密集的发布消息,但是 sub 端可能消费不足。
指令格式:client-output-buffer-limit “,其中 hard表示 buffer 最大值,一旦达到阀值将立即关闭连接;
soft 表示"容忍值”,它和 seconds 配合,如果 buffer 值超过 soft 且持续时间达到了 seconds,也将立即关闭连接,如果超过了 soft 但是在 seconds 之后，buffer 数据小于了 soft,连接将会被保留。
其中 hard 和 soft 都设置为 0,则表示禁用 buffer 控制.通常 hard 值大于 soft。
44. hz 10
Redis server 执行后台任务的频率,默认为 10,此值越大表示 redis 对"间歇性 task"的执行次数越频繁(次数/秒)。"间歇性 task"包括"过期集合"检测、关闭"空闲超时"的连接等,此值必须大于 0 且小于 500。此值过小就意味着更多的 cpu 周期消耗,后台 task 被轮询的次数更频繁。此值过大意味着"内存敏感"性较差。建议采用默认值。
45. include 载入文件
\# include /path/to/local.conf
\# include /path/to/other.conf
额外载入配置文件。
