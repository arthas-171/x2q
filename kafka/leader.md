# kafka 参数
### [go back](/x2q/kafka/kafka)      
### [go home](/x2q)       
  
## 概述
kafka有两个地方涉及到 选择leader
+ 多个broker 选出一个作为leader,负责topic创建与删除,分区和副本管理,代理故障转移
+ 每个 topic的分区 partition,选择一个作为leader partition,其他作为follow,leader partition负责所有读写,并且维护状态信息 HW,LEO(每个副本维护自己的LEO)
leader 根据 LEO算出HW
### HW 和 LEO
+ HW:是指消费者可见的最大的offset,消费者只能拉取HW之前的消息,取一个partition对应的ISR中最小的LEO作为HW
+ 表示每个partition的log最后一条Message的位置

## partition选主  待续
Kafka 的Leader Election方案解决了上述问题，它在所有broker中选出一个controller，所有Partition的Leader选举都由controller决定。controller会将Leader的改变直接通过RPC的方式（比ZooKeeper Queue的方式更高效）通知需为此作为响应的Broker。
https://www.cnblogs.com/mongotea/articles/12246895.html



#### 联系邮箱 xxx_xxx@aliyun.com