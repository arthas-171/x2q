# kafka 参数
### [go back](/x2q/kafka/kafka)      
### [go home](/x2q)       
  
## 首先说一下 describe 显示的topic的属性 如下图
![图片](/static/img/get4.png)
首先最上面5个 
+ topic:topic的名字(废话)
+ partitionCount:分区个数
+ ReplicasFactor:复制因子,如果是2就代表算上本体,一共两个,就是一个本体,一个复制体
+ retention.ms:生命周期,保留多长时间
+ compression.type:压缩类型
再下面就是具体的分区情况
+ partition:分区的号/索引/序列号
+ leader:这个分区leader(负责读写的节点)所在的broker,注意leader负责读写,follow只负责备份
+ replicas:应当拥有的用于备份的节点,不管这个节点上是leader还是follow,也不管这个节点是否存活
+ isr:副本已经同步完成的节点的集合(和leader已经完成同步的),例如图中 partition:2就没有完成同步
## 如何判断副本已经同步
+ 0.9以前 replica.lag.max.messages 指定 leader和follow之间相差的数据条数,这样不太好不好指定
+ 0.9以后 replica.lag.time.max.ms 指定leader和follow之间的相差时间,
如果不满足同步条件,就会被踢出 isr集合,等到完成同步之后再加入
## 分区leader选举,这里说的就是 同一个partition里面有leader和follow.lead挂了如何选出follow
kafka的选举有多重情况,
+ 控制器选举.kafka拥有多个broker但是其中一个会被选举为控制器(kafka controller),负责管理集群中分区和副本的状态,这个依赖于zk
KafkaController会在指定的zookeeper路径下创建临时节点，只有第一个成功创建的节点的KafkaController才可以成为leader，其余的都
是follower。当leader故障后，所有的follower会收到通知，再次竞争在该路径下创建节点从而选举新的leader
+ 分区leader选举,如上文,partition存在一个leader负责读写,isr中存在多个follow备份(最多有多少个取决于配置),**第一步**
从Zookeeper中读取当前分区的所有ISR(in-sync replicas)集合.**第二步**调用配置的分区选择算法选择分区的leader
### 具体的不同
在ISR中至少有一个follower时，Kafka可以确保已经commit的数据不丢失，但如果某个Partition的所有Replica都宕机了，就无法保证数据不丢失了。这种情况下有两种可行的方案：
+ 等待ISR中的任一个Replica“活”过来，并且选它作为Leader
+ 选择第一个“活”过来的Replica（不一定是ISR中的）作为Leader
这就需要在可用性和一致性当中作出一个简单的折衷。如果一定要等待ISR中的Replica“活”过来，那不可用的时间就可能会相对较长。
而且如果ISR中的所有Replica都无法“活”过来了，或者数据都丢失了，这个Partition将永远不可用。选择第一个“活”过来的Replica作为Leader，
而这个Replica不是ISR中的Replica，那即使它并不保证已经包含了所有已commit的消息，它也会成为Leader而作为consumer的数据源（前文有说明，所有读写都由Leader完成）。
Kafka0.8.*使用了第二种方式。根据Kafka的文档，在以后的版本中，Kafka支持用户通过配置选择这两种方式中的一种，从而根据不同的使用场景选择高可用性还是强一致性。
 unclean.leader.election.enable 参数决定使用哪种方案，默认是true，采用第二种方案



#### 联系邮箱 xxx_xxx@aliyun.com