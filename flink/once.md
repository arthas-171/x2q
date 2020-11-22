#  flink end-to-end 恰好一次性
### [go back](/x2q/flink/flink)      
### [go home](/x2q)       

## 概述
如果要做到完全的端到端的恰好一次,需要保证 source源,process处理过程,sink到下游三个地方都支持具体如下
+ source源需要支持保存offset和支持根据offset回放
+ process过程需要支持能够保存checkpoint检查点,保存计算的中间过程和消费数据的offset
+ sink到下游,需要支持幂等性操作,或者支持事务提交(这个也不一定能完美解决)

## source源方面
比如kafka就支持可以重置offset位置,如果下游发生异常可以重新消费这批数据

## process过程
flink基于state支持checkpoint操作,可以设置checkpoint的执行周期,checkpoint操作将把最新的计算的中间结果和消费数据的位置点
记录下来,如果任务失败了从checkpoint检查点中恢复,实现原理基于barrier(栅栏),具体如下
+ 对于不进行shuffle的流,数据源定期插入barrier(栅栏可以理解为一条特殊的数据),下游operator收到barrier的时候进行checkpoint即可
+ 对于需要多流合并的流,下游operator会收到多个流的barrier,但是这些流的速率是不一定相同的,下游task会缓存(buffer)先到的流,等到后到
的流的barrier到达在进行checkpoint

## sink 端一次性
### 幂等写入 例如是一个hbase 覆盖key的value值也无所谓,或者输出结果是个数值,不是明细每次输出这个值都是一样的,覆盖前面的值
### 事务写入 两阶段提交/预写日志
+ 把结果数据先当成状态保存，然后收到checkpoint完成的通知时，一次性写入sink系统 
+ 由于数据提前在状态后端(state backend)中做了缓存，所以无论什么 sink 系统，都能用这种方式一批搞定
+ DataStream API提供了一个模板类：GenericWriteAheadSink来实现这种事务性sink
预写日志 WAL write-ahead-log, 缓存一批,等到checkpoint的时候一次性写入sink,相当于弄一批
但是这样有问题,就是这一批可能写到一半的时候就失败了,导致一部分写入了另一部分没有写入
### flink两阶段提交
两阶段提交two-parse commit
1）对于每个checkpoint，sink任务会启动一个事务，并将接下来所有接收的数据添加到事务里。
2）将这些数据写入外部sink系统，但是不提交它们，只是“预提交”。
3）当它收到checkpoint完成的通知时，才正式提交事务，实现结果的真正写入。
4）这种方式真正实现了exactly-once。
5）这种方式真正实现了exactly-once，它需要一个**提供事务支持的外部sink系统**，Flink提供了TwoPhaseCommitSinkFunction接口。



#### 联系邮箱 xxx_xxx@aliyun.com