# kafka基本概念(未完善)
### [go back](/x2q/kafka/kafka)      
### [go home](/x2q)         
## 什么是消息队列
消息队列就是相当于一个管子,一个人从管子的入口不停的塞(push)数据进去,另一个人不停的从出口拉取(pull)数据
## kafka 相关概念/术语
**综述** 一个kafka集群可以包含多个producer,多个broker,多个consumer,每个producer可以对应多个topic,
每个topic可以有多个partition,但是每个consumer只能属于一个consumerGroup
+ producer:生产者,往kafka里面写入数据,这个写入一般采用异步分批次写入的方式,有一个缓冲区,
有两个参数linger_ms(批次时间,默认0秒)/batch_size(批次大小,默认16384 Bytes),满足其一就发送
+ consumer:消费者,消费者采用pull(拉取)的方式去kafka拉取数据消费,自己控制速度和维护offset
+ consumerGroup:消费者组,一个consumer只能属于一个消费者组,消费者组统一维护消息消费的offset,
一个topic中的消息只能被consumerGroup中的一个consumer消费,因此
+ + 单播:所以的consumer属于同一个consumerGroup,单播就是一条消息只有一个consumer消费掉
+ + 广播:每个consumer单独一个consumerGroup,广播就是一条消息被每个consumer消费一次
+ topic:就是**主题**,同一类型(指的是业务上同一个类型)的消息,放到一个topic里面,这个是一个逻辑概念
一个topic可以包含多个partition,每个partition内部消息是有序的,kafka可以根据消息数据的key,把相同的
key的数据放入同一个partition,如果没有key就轮询放入partition
+ partition:物理上的概念,本质上是存储数据的日志文件的目录,一个topic可以包含多个partition,partition内部消息有序,partition越多broker写入的并行度越高
,并且下游消费的并行度也会提高,如果是spark streaming,topic的partition数决定起始的task数量
+ offset:每条数据的编号,partition中的每条数据都会按照时间顺序递增分配一个编号,消费者的offset有两种保存方式
取决于api,对于使用zookeeper连接的会保存在zk上面,对于新版api通过brokerserver连接的会保存在kafka自带的topic"__consumer_offsets"下面
+ broker:消息的处理节点,一个broker就是一个kafka server的节点,一个kafka集群包含多个broker
## kafka 常用配置
### producer端参数
+ ack producer 向kafka集群发送数据,确保数据到达的机制
+ + ack=1,leader节点接收到数据即认为成功
+ + ack=0,不考虑kafka集群是否接收成功,只要发送不报错就行,不等broker确认
+ + ack=-1,leader接收成功并且follow同步成功才认为成功
+ batch.size=16384(Bytes) producer批量发送的基本单位，默认是 16384 Bytes  lingger.ms和batch.size满足其一就发送 
+ linger.ms=10(秒,默认0秒) producer sender线程检查batch批次是否具备的时间间隔
+ buffer.memory=33554432(byte) 可以使用的最大内存来缓存等待发送到server端的消息 byte
+ retries=0 producer发送失败重试次数
+ enable.idempotence = true,producer端的保证恰好一次,这种是设置producer的幂等性,每个producer在初始化的时候都会被分配一个唯一的Producer ID，
producer向指定topic的partition发送消息时，携带一个自己维护的自增的Sequence Number。
broker会维护一个<pid,topic,partition>对应的seqNum。 每次broker接收到producer发来的消息，
会和之前的seqNum做比对，如果刚好大一，则接受;如果相等，说明消息重复;如果相差大于一，则说明中间存在丢消息，拒绝接受。
### consumer端参数
+ enable.auto.commit:若设置为true consumer在消费之前提交位移 就实现了at most once
若是消费后提交 就实现了 at least once 默认的配置就是这个
+ auto.offset.reset:
+ + earliest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费
+ + latest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据
+ + none:各分区都存在已提交的offset时，从offset后开始消费；只要有一个分区不存在已提交的offset，则抛出异常
+ consumer端的恰好一次性需要,自己管理offset,自己定义一个两阶段提交,先缓存再提交

### flink 消费者是pull 拉模式
好处如下
+ 自主决定获取消息的批量和频率
+ 避免推模式中的一些问题：如果使用push模式，生产者可能无法准确知道消费者的消费能力，难以控制推送速度
+ 适合大规模分布式系统：pull模式在分布式系统中表现更好，因为消费者可以自主地从一个或多个broker中拉取消息，无需等待broker的推送。这使得Kafka能够在大规模分布式系统中实现高效的消息消费。



#### 联系邮箱 xxx_xxx@aliyun.com