#  flink 基础
### [go back](/x2q/flink/flink)      
### [go home](/x2q)       

## 文档建议 一个taskmanager的slot数应当等于taskmanager运行的jvm所在机器的cpu核数

## flink的分布式流计算
本质上讲,flink的流是一个分布式的流,每个算子代表一个类型的task,这个task是在多个机器或者容器的多个线程中并行执行的,
如果涉及到 shuffle 则在网络间进行数据交换,如下图
![图片](/static/img/get4.PNG)  

## flink JobManager和 TaskManager 
flink 也是采用主从结构,jobmanager是master节点,taskmanager是slove节点(准确的说应该叫worker更合适),他们之间通过
Akka Framework (一款高性能、高容错性的分布式并发应用框架。akka底层采用scala语言实现，并基于actor并发模型，
天然拥有异步、分布式能力，且具有很好的并发性能和容错机制) 通讯,

### JobManager
JobManagers 协调分布式计算。它们负责调度任务、协调 checkpoints、协调故障恢复等。
每个 Job 至少会有一个 JobManager。高可用部署下会有多个 JobManagers，其中一个作为 leader，其余处于 standby 状态。
### TaskManager
TaskManagers（也称为 workers）执行 dataflow 中的 tasks（准确来说是 subtasks ），并且缓存和交换数据 streams。
taskmanager 是一个jvm里面的进程,一个jvm里面一般只有一个taskmanager进程,一台机器(或者容器)上一般只运行一个jvm,
但是taskmanager下面会有多个slot插槽每个插槽可以运行一个task,每个task本质是一个线程,同一个taskmanager下的task
可以同享机器cpu,但是不可以共享内存,taskmanager会将内存按照slot数均分,每个slot之间内存是隔离带
**文档建议 一个taskmanager的slot数应当等于taskmanager运行的jvm所在机器的cpu核数**
![图片](/static/img/get5.PNG)  
Tasks 在同一个 JVM 中共享 TCP 连接（通过多路复用技术）和心跳信息（heartbeat messages）。
它们还可能共享数据集和数据结构，从而降低每个 task 的开销
![图片](/static/img/get6.PNG)  
## 状态后端 status backend
jobmanager 负责触发 checkpoint和 checkpoint ack(结果状态验证), 具体的保存checkpoint(快照)是taskmanager做的
taskmanager会根据配置的状态存储方式(内存/hdfs/RocksDB)把自己的状态存起来
![图片](/static/img/get7.PNG)  
#### 联系邮箱 xxx_xxx@aliyun.com