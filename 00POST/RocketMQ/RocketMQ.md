
# MQ最主要的三个方面:
1.异步,提高系统的响应速度和吞吐量
2.解耦,减少服务之间的影响,提高系统的稳定性
有利于消费的分发,消费者将消息丢到消息队列,不同的服务者可以从同一个队列里面拿到自己的消息
3.削峰，例如，上游的服务突然激增,可以暂存部分消息,减少下游服务的压力


# 如何选择使用哪个MQ?
1. Kafka:
	- 优点：性能非常好,集群高可用
	- 缺点：数据会丢失,功能比较单一
	- 适用于数据量大,允许部分数据丢失的情况，如：日志分析,大数据采集
2. RabbitMQ:
	- 优点：消息可靠性高,功能全面
	- 缺点：吞吐量低,消息容易积压,基于erlang语言不好定制
3. RocketMQ:
	- 优点：高吞吐量,高性能,高可用,功能比较全面
	- 缺点：客户端只支持java


如何保证消息不丢失?

首先最容易想到的一点应该是跨网络的情况

看图片下面是几个跨网络的情况,我已经圈出来了

![](https://i0.hdslb.com/bfs/note/1ee2c5f736caa8034eef86d26c85dbd08ccef008.png@808w_!web-note.webp)

除了跨网络的情况之外,还会存在消息丢失,丢失的地方在于内存同步到硬盘,比如突然断电,内存上的消息还来不及同步到硬盘,这也会造成消息丢失

![](https://i0.hdslb.com/bfs/note/156d702c33e739375344957efc8f58ee9e6688c7.png@808w_!web-note.webp)

  

总结上面的几个情况:
1.消息发送到MQ
2.消息主从同步
3.消息消费
4.消息存盘

  

这些环节要如何保证消息不丢失呢?

1.发送消息不丢失,如何保证呢

Kafka:消费者发送消息,Kafka会返回一个确认消息,这个确认消息会执行消费者的回调函数,如果回调函数没有执行成功,消费发送失败

消费失败,进行消息重试

  

RocketMQ:对于功能更强一些, 上面的Kafka注册回调方法的方式肯定时支持的
RocketMQ更牛逼的地方,可以使用分布式事务机制
需要保证生产者,消费者端的事务要么同时成功,要么同时失败
这种情况的话很复杂,RocketMQ也不能保证

![](https://i0.hdslb.com/bfs/note/c557f93fc57cd1a8b0b64ff63b93b29b0291b53d.png@882w_!web-note.webp)

但是RocketMQ能够保证的是什么

下面如图所示,RocketMQ能保证的是1,2两个操作的原子性

![](https://i0.hdslb.com/bfs/note/d271610688b87ca91184306654f36c85e2bee0b1.png@882w_!web-note.webp)

这种方式我们只需要保证的是啥

只需要保证,只要我们的消息发送到MQ成功之后

我们才会让本地事务成功执行

对于消费者来说也是一样,我们只需要保证消息成功被消费,才会提交本地事务

![](https://i0.hdslb.com/bfs/note/ec355c8b5cc386a330d942f7b12cfdfe6bce6181.png@882w_!web-note.webp)

  

大概的流程

1.生产者发送消息确定MQ存在

2.生产者发送具体的消息,以及本地事务的状态

上面的流程结束后会遇到下面几种情况

本地事务成功 本地事务失败 本地事务未知

本地事务成功->MQ才会将消息推送给消费者,消费者才能成功拉取

  

本地事务失败->那么消息会直接被MQ丢弃

  

本地事务未知->MQ会向生产者发送消息回查,生产者接收到消息回返回本地事务状态

  

还有一点需要注意的是:生成者确定MQ存活之后才会执行本地事务

![](https://i0.hdslb.com/bfs/note/ab9ab63ecf0bab7393bfb01a2ffeed99452aeb09.png@882w_!web-note.webp)

  

  

好吧,既然这里提到了一种场景,那我们就引入一下

视频中说用事务消息可以非常轻松的应付订单超时的情况

为啥可以这么说,我们分析分析,分析过程如下:

生成者下订单,生产者通过本地事务存订单,往MQ发送消息的时候,事务的状态设置为未知的状态

欧克欧克,现在注意->上面我们已经了解到,MQ对于这种未知状态会主动发起回查,查询本地事务的状态,这里不是去查询本地事务的状态,而是去查询支付系统,比如说订单5分钟之后未支付,我们就会自动取消,配置MQ去查支付系统,5分钟内完成15次回查,每隔一段时间我们去查一下支付系统,五分钟都没有查询到支付信息,那么MQ就会丢掉此次创建订单的消息,下游服务就不能将订单成功创建

  

好,说完了我们上面的

我们继续说RabbitMQ

RabbitMQ也是可以通过消息发送,MQ执行回调来确保消息不丢失的

除此之外,RabbitMQ也可以进行事务消息的发送,只不过使用的是手动事务,RabbitMQ使用的是通过channel的方式来做的,每次客户端和MQ之前开启衣蛾channel都是阻塞的,在开启事务和关闭事务之间,这个channel是不能用于做其他事情的

![](https://i0.hdslb.com/bfs/note/8e9313d74b1cb66593c1f46322708001d9854ff3.png@882w_!web-note.webp)

  

Publisher Confirm,整个处理流程跟RocketMQ的事务消息基本上是一样的

  

P4

上面所了消息发送时保证不丢失

下面说一下消息主从同步如何保证不丢失

有两种方式->同步同步,异步同步

同步同步的方式会产生阻塞,原因是我们的消息回调发生在同步之后,由我们的从节点发送消息回调请求

![](https://i0.hdslb.com/bfs/note/79dda463f970161ac5b14b512598f02a3bfed934.png@882w_!web-note.webp)

  

另外一种就是,异步同步

这种方式就不会阻塞,因为消息回调任然是由我们的主节点来发送回调的,并且区别上面同步同步的地方在于,当MQ执行回调的时候,会异步的将消息同步到从节点上

![](https://i0.hdslb.com/bfs/note/8cd84ea6864422301852aca33dd891877eb4d434.png@882w_!web-note.webp)

这样的话,没有阻塞, 并发量会更高了

注意的是:异步同步可能会有丢消息的风险,所以要保证消息的不丢失还得是使用同步同步的方式,因为同步同步的方式如果没有同步成功,期间发生了消息的丢失,那么回调就不会成功,回调不会成功,那么生产者就知道自己的消息没有发送成功

  

注意一点:这里的消息丢失是不可以避免的,只不过我们可以通过某些设置来告知生产者,咱们这消息到底发没发送成功,这才是保证消息丢不丢失的主要一件事情

  

上面是针对MQ的普通集群

  

下面是针对MQ的Dledger集群,两阶段提交,并且主节点会一直被重新选举,当从节点大部分同步成功,才会将回调请求返回生产者

  

我们在来说说RabbitMQ的消息同步:

![](https://i0.hdslb.com/bfs/note/badc157ec4bbd6bcc34fe15990122bbafff66093.png@882w_!web-note.webp)

RabbitMQ中有镜像集群:节点之间会主动进行数据同步

所谓的同步同步, 指的是将数据同步到服务端之后咱们才发送回调请求

没有继续解释Kafka了, 一般Kafka都是用在允许少量消息丢失的场景

  

MQ如何保证消息刷到磁盘上的时候消息不丢失呢

1.同步刷屏 写一个消息,立即存盘

RocketMQ中支持同步刷盘,也支持异步刷盘,同步刷盘指的是写到内存中,然后立即将消息写到磁盘,但是效率会更低,因为IO操作比较多

异步刷盘,大概我猜测应该是这样的,先将多条消息写到内存,等到满足一定条件,再批量的一次性刷到磁盘里面

  

说完RocketMQ,我们继续说RabbitMQ

把一个队列设置成一个持久化队列,写到内存的同时就写到磁盘

  

okok

下一个保证消息消费不丢失,因为消费的时候是有重试机制的,消费成功了,才会移动偏移量,如果消费失败了,或者根本没有消息

![](https://i0.hdslb.com/bfs/note/08c67dfc80836cf6e8f05c3eff18d8a901809353.png@882w_!web-note.webp)

进行同步的消费,这样的话才能保证消息不会丢失

到底丢没丢失,其实主要是通过下游往往上游发送确认请求,只有同步的发送了确认请求,那么这个消息才算发送成功了

对于RocketMQ:使用默认的方式消费就可以了,不要采用异步的方式

1.MQ是一个队列,一个一个消息在里面排列着

2.消费者消费完了

3.执行本地事务

4.MQ中的偏移量就会移动

5.

为啥RocketMQ采用异步的方式可能会造成消息丢失呢?

因为当消费者拿到消息之后,直接将消息的确定认请求发送给MQ,本地的事务可能会执行失败,但是MQ又不会进行消息重试,那么对于这条消息的处理就会漏掉

说了这么多,还是要再次强调一点,避免消息丢失不是为了认消息真正的不消失在系统中,主要的一点应该是,不能漏掉消息的处理

对,更合理的解释,应该是不能漏掉消息的处理,其实是允许消息在这些过程中丢失的,只不过丢失了,需要进行消息的重试,确保消息不会在系统的传输过程中漏掉

  

下面的方式,是消费者异步方式消费MQ中的消息,如果当我们直接将消息的确认消费成功发送给MQ,MQ就会移动偏移指针,MQ里面就会把这条消息当做已经处理过的消息,但是起始在消费者的本地事务里面

![](https://i0.hdslb.com/bfs/note/50c8e560c0d872d05e39f38028adfe7df5629dba.png@882w_!web-note.webp)

  

P5

如何保证消息的幂等性?

如何保证消费者的重复消费?

为啥会出现重复消费?

哪些情况会出现重复消费?

MQ超时了都没获取到消费者消费成功的报文,那么MQ会触发重试机制,那么MQ就会再发一次消息发给消费者

消费者就会重复拿到两次的消息,然后执行两次的处理逻辑

ok,上面就是一种重复消费的场景

这种重复消费的场景应该怎么解决呢,首先应该交由消费者自己来解决

好吧先说一种,RocketMQ特有的机制,但是kafka和RabbitMQ没有的机制

就是RocketMQ中每条消息都有一个messageID,而这个messageID就可以提供给消费者判断是不是重复的消息

消息者那边说,哎,这条消息我已经成功处理完了,我把它存下来,表示我已经处理过了

ok,下次MQ再推这条消息过来,消费者一查,害,这条消息不是我已经处理过的吗,ok不用在过一遍业务逻辑了

但是RocketMQ的messageID是全局唯一的吗? 不是的

所以这种方式也不一定是一个好方式

那么最好的方式是什么呢?

自己在消息中自己带一个有业务标识的全局唯一的ID

P6

如何保证消息的顺序呢?

RocketMQ可以保证消息局部有序,

啥是局部有序呢?

举一个列子,就比如,我们有一个订单,这个订单的消息顺序是不能打乱的,必须保证消息从下往上消费

简单的概况就是,对于这个订单来说,我得保证这个订单的数据是全局有序的

对于两个订单,1号订单里面的消息顺序是需要保证的,2号订单里面的顺序也是需要保证的

但是1号订单和2号订单的消息相互之间是可以打乱的,就不用保证全局有序

保证的有序,仅仅只是单个订单里的消息时有序的

  

要保证这种局部有序性,生产者消费者端都要有一定的机制的,也就是说生产者消费者都需要设置相关的配置

  

  

要知道, 我们只用一个MQ,本身队列就可以保证顺序的啊

如果我们往这个队列里面发送消息,队列里面的顺序就是我们的发送顺序

所以,为啥需要保证消息的顺序啊

天生就是有顺序的!

![](https://i0.hdslb.com/bfs/note/bd50f76beb9320a3e7b94521fd8d03d54d9a2922.png@882w_!web-note.webp)

  

欧克欧克,那就来说说会出现顺序打乱的情况

为啥会出现消息顺序打乱的情况

分布式的情况下,一个Topic下有多个队列,那么就我们刚刚说的情况,一个订单会对应多条消息,而且这些消息在业务上必须具有顺序处理的性质

可是因为一个Topic下面有多个队列,在进行消息分发的时候,我们将消息分发到不同的队列里面,消费者在拿消息的时候,拿到这个订单对应的消息,这些消息的顺序性就不能得到保证了

![](https://i0.hdslb.com/bfs/note/ae48216cbe3c249417191b0af1668b3451cb7c0a.png@882w_!web-note.webp)

上面已经说了, 生成者丢到MQ里面是随便往队列里面丢的,消费者拿的时候就不能保证顺序性了

并且消费者会并发的从队列里面拿消息,多个消费者会拿到同一个订单的不同消息,这些消息的处理顺序会有保证吗,哪些消费会先处理,哪些消息会后处理,是没有保证的

如图所示,消费者会并行的从每个队列拿消息

并且同一个订单的几条消息,哪些会先执行,哪些会后执行,也是没有保证的

![](https://i0.hdslb.com/bfs/note/9184bd782f3550988c390ab990d63e28d11ac190.png@882w_!web-note.webp)

  

okok,上面我们说了一下问题的背景, 就是为啥会出现消息顺序,消费的时候顺序是没法保证的

说了这么多,其实还是和业务相关,因为对于这个订单的消息,处理顺序必须有序,但是在送到MQ的过程中可能会出现消息被打乱的情况,这个问题需要解决

另外就是,从MQ里面拿消息,因为拿消息的方式,导致消费者在处理时对于同一个订单的消息,处理顺序也是未知的会被打乱的,那么我们依旧需要在这里保证消息不能被打乱,特别指的是这种分布式情况下,有多个消费者实例的情况

  

说了这么多,具体是如何保证消息的顺序的呢?

first,对于同一组消息,生成者要把消息放到同一个队列中,这里就解决了,上面生成者放到Topic下面不同队列导致顺序打乱的问题

next,消费者每次消费的时候,是消费整个队列中的消息,而不是并行的从不同的队列中拿消息,并行的拿消息,会导致消息分散到不同的消费者中去,而这些消费者的处理,又不知道哪些消费者先处理消息,哪些消费者后处理消息

![](https://i0.hdslb.com/bfs/note/468170ae22a160d3a7b9288a8a8947e84b65b514.png@882w_!web-note.webp)

RocketMQ中对上面的问题有完成的处理,但是另外两个RabbitMQ和Kafka中,就没有这样的设计

  

那对于Kafka和RabbitMQ中没有这样的设计,我们应该做什么来保证消息的顺序呢?

其实贼简单,就是放弃多个队列,放弃分布式消费者集群

RabbitMQ, 保证MQ中间件exchange只对应一个队列,那么生成者往一个队列里面丢消息,消息就不会被打乱了,并且这个队列只能对应一个消费者

  

对于Kafka,生产者通过定制partition分配规则,将消息分配到同一个partition,Topic下只对应一个消费者

  

P7

如何保证消息的高效读写?

啥问题,这是?

首先要搞懂啥是零拷贝

传统的文件拷贝:

![](https://i0.hdslb.com/bfs/note/0e84a22b0236d9c9203a690ab81ae651e5612189.png@882w_!web-note.webp)

  

而零拷贝就是针对传统的拷贝进行优化

零拷贝:

零拷贝优化的点在哪里?->就是拷贝的时候不用在用户空间进行拷贝,直接在内核空间完成拷贝

![](https://i0.hdslb.com/bfs/note/3a44c003f381c3813921d2d14f2691ba41199e0a.png@882w_!web-note.webp)

消息中间件中是如何使用零拷贝技术的呢?

RocketMQ当中对应文件的读写,就是通过mmap的方式(零拷贝)对文件进行读写,比如commitlog(一个生成的文件)就是在内核就拷贝出来的,并且一次就生成一个G的文件

  

对于kafka,使用DMA的方式将硬盘数据加载到网卡,所以这个效率是非常快的

  

P8

MQ如何保证分布式事务的最终一致性?

![](https://i0.hdslb.com/bfs/note/7fcfd426b614e36442176be3a8710294c1be932b.png@882w_!web-note.webp)

贼简单,其实前面讲消息丢失,消息幂等性,就已经说了,需要保证两点:

1.上游的生产者要保证100%投递成功,其实就是事务消息,起面已经说,因为事务消息就不会出现消息丢失的情况,消息丢失会进行消息重新

2.消费者的消费需要保证幂等消费,唯一ID+业务自己解决

  

总结,保证最终一致性,其实就是解决消息丢失问题,以及消息重复消费问题,OK,解决这两个问题就行了,一种方法是使用事务消息,一种方式是使用全局唯一ID来保证幂等性

  

P9

如何设计MQ?

不要放飞自我漫无目的(感觉就是见得少了)

纠结一下次要的细节

  

好一点的方式:

整体到细节

业务场景到->技术细节

以现有场景为基础,比如RocketMQ,因为RocketMQ是后来者,借鉴了前两个的一些思路

  

首先实现一个单机的队列结构,支持高效,可扩展,支持扩容收容

将单机的队列设置成分布式队列,分布式集群管理

消息发送到Topic,采用轮洵的方式丢到分布式队列

如果采用定制的顺序消息,可以怎么样,设置

节点怎么设置,一个消费者消费一个队列,消费者和队列的对应关系

  

按照顺序消费

按照链路消费

![](https://i0.hdslb.com/bfs/note/6a49107f774ec715260f47aeff47ee55d92e16a4.png@882w_!web-note.webp)

P10

RabbitMQ

的负载均衡和广播消息

负载均衡就是队列在分发消息给消费者的时候,一个队列可以将队列中的消息分发到多个消费者

RabbitMQ的广播消息:其实也会简单,就是生产者将消息丢给MQ,MQ会将消息发送到多个队列里面,每个队列都对应一个消费者,这样就实现了广播消息

RabbitMQ里面有一个交换机,生成者首先将消息丢给交换机,交换机在路由到一个或者多个队列中,如果是广播消息就是路由到多个队列,如果路由不到,就会返回给生产者,直接丢弃

  

RabbitMQ里面有多个交换机,多个队列,而交换机和队列的关系是由Binding来维护,相当于维护了一张映射表多对多的关系

  

消息会指定一个路由Key,然后交换机会通过BindingKey投递选择队列,选择对应的队列

  

P11

RabbitMQ如何保证消息一定会被发送?消息一定会被接收?

也就是我们之前说的消息不会丢失,消息一定会被发送,指的是消息发送从生产者发送到MQ一定不会出现消息被漏掉的情况

消息一定会被接收,消息不会被丢失,指的是MQ发送到消息者端,消息一定不会漏掉的情况,消息一定要经过消费者处理完成之后,MQ才会将消息从MQ里面删掉

  

对于RabbitMQ,他有一个确认机制:

发送方有个一确认机制,接收方也有一个确认机制

首先说一下发送方的确认机制

信道可以设置为confirm模式,所有的信道上发布的消息都会分配一个唯一ID

如果RabbitMQ收到消息了,但是没有投递到信道里面(队列),会返回一条失败的消息给生产者

  

生产者会有两个回调接口,一个是MQ成功收到消息的,一个是MQ没有成功收到消息的,生产者一定会执行其中一个回调函数,生产者发送消息既可以是同步的也可以是异步的,同步的需要等待MQ返回确认信息,异步的不用等待确定信息,但是最终都会触发成功或者失败的回调函数,只有这样,对于生产者来说,消息有没有投递成功才是可以知道的

  

消费放接收的确认机制:

默认情况下MQ可以指定一个noAck参数,表示是否要打开确认机制,默认情况下noAck是为true的,也就是说,MQ只要将消息投递,就会直接将消息从队列里面删除,不管接收方有没有收到消息

上面这种方式是自动提交,消息一投递出去就马上删除了

如果是自动提交,消息投递出去还需要等待消费者返回了ack

这里需要注意一点,如果MQ将消息发送出去了,但是消费放没有返回Ack,会导致MQ阻塞,并且不会给这个ack设置超时时间,也就是说MQ不会因为消费者处理消息耗时太长进行消息删除或者消息重试,只有和消费者连接失败之后才可能进行消息重试

这样设置的目的是啥?

一方面,可以起到限流的作用,如果这一条消息没有处理完,那么后面的消息就不会发送给其他消费者

另一方面,对于消费者来说他处理消息可以很长很长一段时间去处理,保证最终一致性

  

如果消费者返回ack之前断开了链接,那么rabbitMQ会重新将消息发送给下一个消费者,也就是说断开链接之后,rabbitMQ会对消息进行重新投递,但是这里也有一个问题,断开链接的时候可能也会投递了一条消息给消费者,现在重新投递,就相当于投递了两次,并且前一次的投递产生了效果,那么就出现了重复消费的情况,可以通过设置全局唯一消息ID来判断,是不是已经处理过的消息,over~

P12

RabbitMQ的事务消息?

事务消息指的是,发送这条消息和发送完这条消息要进行一些处理,发送消息和事物处理都要保持一致性,发送成功了才会提交事务,发送失败了就会回滚事务

最主要的区别在与,事务消息可以保证本地逻辑和消息发送的原子性,如果本地事务执行有错,也可以让消息的发送回滚回来,如果消息发送出错,也可以让本地事务回滚回来

  

MQ里面会维护一个队列,专门用来处理事务消息,先将消息发送到这个队列,只有等到本地事务提交了,才会将消息送到后面绑定消费者的队列里面

P13

死信队列,rabbitMQ里面的

相当于给这种发送失败了的消息做的一个兜底

  

如果消息发送失败,就会将消息丢到死信队列里面

如果消息在队列中的时间超时,会被丢到死信队列

如果消息队列已经满了,也会被投递到死信队列中

  

RabbitMQ的延时队列

很简单,就是给队列设置一个TTL,等到消费时间到了,消费者才可以从延时队列里面拿消息

或者可以给一条消息的最大存活时间

  

如果设置在队列上面,那么进入队列时,所有的消息都会被设置TTL

如果消息有TTL队列也有TTL,值小的那个会被使用

延时队列是怎么实现的呢?

消费者监听是直接监听延时队列吗?

不是,消费者可不是直接监听延时队列,其实就是普通队列可以设置超时时间,当时间到达TTL后,会将消息丢到死信队列,其实消费者直接监听的是死信队列