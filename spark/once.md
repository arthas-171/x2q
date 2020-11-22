# spark streaming+kafka对恰好一次的支持
### [go back](/x2q/spark/spark)      
### [go home](/x2q)        
## 概述
也是分成 三个方面 一是 读取数据,二是spark streaming处理过程,三是写入下游
## 读取数据 有两种方式  receiver(接受者方式)和 direct(直接方式)后者比较好
### receiver(接受者方式)
+ 基于receiver的采用kafka高级消费者API
+ 每个executor进程都会不断拉取消息，并同时保存在executor内存与HDFS上的预写日志（Write-Ahead log，WAL）
当消息写入WAL后，自动更新ZK中的offset
### direct(直接方式)
+ 基于direct的方法采用Kafka的简单消费者API。
+ executor不再从Kafka中连续读取消息，也消除了receiver和WAL。Kafka分区与RDD分区一一对应
+ driver线程只需要每次从Kafka获得批次消息的offset range(offset 范围 其实offset和截止offset 因为spark streaming是微批次)
然后executor进程根据offset range去读取该批次对应的消息
+ 由于offset在kafka中能唯一确定一条消息，且在外部只能被Streaming程序本身感知到，因此消除了不一致性，达到了Exactly Once
+ spark streaming自身可以借助第三方缓存等保存offset

## spark streaming checkpoint过程
spark streaming 通过定期设置检查点,来保证程序可以从失败中恢复过来,
### **但是这里有两个个问题**,
第一个是 spark streaming的checkpoint过程,由driver
发起,但是driver下发task给executor执行和checkpoint操作不是原子性的,也就是说可能task执行失败,但是checkpoint中报错offset已经消费成功
如下图
![图片](/static/img/get.jpeg)
                    
第二个是于 Spark 处理的数据单位是 partition 引起的。比如在处理某 partition 的数据到一半的时候，由于数据内容或格式会引起抛异常，
此时 task 失败，Spark 会调度另一个同样的 task 执行，那么此时引起 task 失败的那条数据之前的该 partition 数据就会被重复处理，
虽然这个 task 被再次调度依然会失败。若是失败还好，如果某些特殊的情况，新的 task 执行成功了，那么我们就很难发现数据被重复消费处理了                    

## 针对写入下游  一般两种方式  幂等和二阶段提交
+ 幂等需要下游支持幂等操作,比如hbase 重入写入rowkey会覆盖(准确的说是会有版本)
+ 采用二阶段提交 先预写日志,在提交,但是也不能完全保证成功,如果需要完全保证需要下游接受的源,支持事务操作
#### 联系邮箱 xxx_xxx@aliyun.com