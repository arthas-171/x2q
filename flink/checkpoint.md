#  flink 中的容错 checkpoint/savepoint
### [go back](/x2q/flink/flink)      
### [go home](/x2q)       

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

#### 联系邮箱 xxx_xxx@aliyun.com