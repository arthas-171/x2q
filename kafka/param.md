# kafka 参数
### [go back](/x2q/kafka/kafka)      
### [go home](/x2q)       
  



## producer端参数
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

##consumer端参数
+ enable.auto.commit:若设置为true consumer在消费之前提交位移 就实现了at most once
若是消费后提交 就实现了 at least once 默认的配置就是这个
+ auto.offset.reset:
+ + earliest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费
+ + latest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据
+ + none:各分区都存在已提交的offset时，从offset后开始消费；只要有一个分区不存在已提交的offset，则抛出异常
+ consumer端的恰好一次性需要,自己管理offset,自己定义一个两阶段提交,先缓存再提交


#### 联系邮箱 xxx_xxx@aliyun.com