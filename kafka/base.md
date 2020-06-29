# kafka基本概念
### [go back](/kafka.md)      
### [go home](../README.md)     
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
+ offset:每条数据的编号,partition中的每条数据都会按照时间顺序递增分配一个编号
+ broker:消息的处理节点,一个broker就是一个kafka server的节点,一个kafka集群包含多个broker
## kafka高性能的日志存储机制
## kafka高可用的副本机制
## kafka日志文件写入过程
## broker如何协同工作




#### 联系邮箱 xxx_xxx@aliyun.com