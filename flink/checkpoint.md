#  flink 中的容错 checkpoint/savepoint 和重启策略
### [go back](/x2q/flink/flink)      
### [go home](/x2q)       

## checkpoint 原理
   首先checkpoint指定触发间隔后,jobManger会指示每个source源都会定期的生成barrier(栅栏),他是一条特殊的数据和水位线类似,在流中的位置无法提前或者延后
   ,barrier把无限的流分成了若干段,之后有两种情况一种是没有涉及到shuffle过程的operation
   当barrier到达时就会对先于barrier的数据的状态等进行一次记录到检查点,后面的数据会记录到下一次检查点,如果涉及到shuffle,需要接受上游多个barrier
   但是barrier不是同时到达的,这个时候就需要等待所有barrier都到达,然后对齐时间时刻,再进行一次检查点操作,
但是这里有一个情况就是先到达的barrier所在的task,在等待的时候会继续处理数据, Operator会将数据记录（Outgoing Records）发射（Emit）出去，
,之后根据状态后端的不同,这些检查点会全量或者增量(如果是rockDB)的保持在一个hdfs目录上,

## checkpoint是分布式task快照
+ checkpoint,需要主动开启,默认是不开启的
可以进行如下配置
                                               
                                               
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment()

// start a checkpoint every 1000 ms 周期
env.enableCheckpointing(1000)

// advanced options:

// set mode to exactly-once (this is the default)  设置检查点的语义
env.getCheckpointConfig.setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE)

// make sure 500 ms of progress happen between checkpoints  每两次checkpoint的时间间隔
env.getCheckpointConfig.setMinPauseBetweenCheckpoints(500)

// checkpoints have to complete within one minute, or are discarded 每次执行checkpoint的超时时间
env.getCheckpointConfig.setCheckpointTimeout(60000)

// prevent the tasks from failing if an error happens in their checkpointing, the checkpoint will just be declined.
// 如果执行checkpoint的时候失败,不会影响到task失败,只是checkpoint被拒绝
env.getCheckpointConfig.setFailTasksOnCheckpointingErrors(false)

// allow only one checkpoint to be in progress at the same time  允许多少个checkpoint并行执行
env.getCheckpointConfig.setMaxConcurrentCheckpoints(1)

// enables the experimental unaligned checkpoints  不等待 barrier对齐
env.getCheckpointConfig.enableUnalignedCheckpoints()
```                                               

## flink的重启策略
   **全局设置**默认的重启策略是通过Flink的flink-conf.yaml来指定的，这个配置参数restart-strategy定义了哪种策略会被采用。
如果checkpoint未启动，就会采用no restart(不重启)策略，如果启动了checkpoint机制，但是未指定重启策略的话，就
会采用fixed-delay策略，重试Integer.MAX_VALUE次,具体的策略如下
+ Fixed delay:固定次数重启,这种就是如果失败了会固定重启多少次,如果超过了这次数任务就彻底失败了
+ Failure rate:失败率重启,如果任务失败了,就会重启,如果重启失败次数超过了一定概率,就彻底失败
+ No restart:不重启
### 在代码中针对单个job设置
+ 无重启
```scala
val env = ExecutionEnvironment.getExecutionEnvironment()
env.setRestartStrategy(RestartStrategies.noRestart())
```
+ 固定次数
```scala
val env = ExecutionEnvironment.getExecutionEnvironment()
env.setRestartStrategy(RestartStrategies.fixedDelayRestart(
  3, // 重启次数
  Time.of(10, TimeUnit.SECONDS) // 延迟时间间隔
))
```
+ 失败率
```scala
val env = ExecutionEnvironment.getExecutionEnvironment()
env.setRestartStrategy(RestartStrategies.failureRateRestart(
  3, // 每个测量时间间隔最大失败次数
  Time.of(5, TimeUnit.MINUTES), //失败率测量的时间间隔
  Time.of(10, TimeUnit.SECONDS) // 两次连续重启尝试的时间间隔
))
```


## todo  checkpoint 和 savepoint的区别
https://zhuanlan.zhihu.com/p/79526638
#### 联系邮箱 xxx_xxx@aliyun.com