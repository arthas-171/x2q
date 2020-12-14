# kafka 高水位
### [go back](/x2q/kafka/kafka)      
### [go home](/x2q)         
## 概述
kafka 高水位有两个作用
+ 标识消息可见性,标识分区下那些消息是可以被消费的
+ 帮助kafka完成副本同步
## kafka  HW和LEP 和 ISR
+ HW:就是 high watermark,是指消费者可见的最大的offset,消费者只能拉取HW之前的消息
+ LEO:就是 log end offset,当前分区最后一条offset+1,分区 ISR 集合中的每个副本都会维护自身的 LEO ,
而 ISR 集合中最小的 LEO 即为分区的 HW
+ ISR:是in-sync replica,已经完成同步的副本


#### 联系邮箱 xxx_xxx@aliyun.com