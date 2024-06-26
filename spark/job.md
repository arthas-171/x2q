# Spark 任务提交运行基本概念
### [go back](/x2q/spark/spark)      
### [go home](/x2q)       
## 基本概念

+ application:整个spark应用程序
+ driver:相当于驱动节点,负责资源申请,任务分配和监控,也即是运行main函数所在进程的节点,main函数中会创建 Sparkcontext(Spark上下文),它会包含spark env(spark 运行环境等),并且和 ClusterManager 通讯,申请资源 分配任务 监控任务,sparkContext中创建的类主要包括spark env, TaskScheduler(任务调度器), DAGScheduler (DAG调度器)
+ executor:应用运行在worker节点(这里只的是物理节点,一个机器上的一个jvm)上的进程,该进程负责运行一些task,并且负责将这些task的运行过程中的数据存储在内存或者磁盘上,executor会将task包装成 taskRunner,然后从线程池中分配一个空闲的线程运行它,executor能并行运行多少个task,取决于给他分配的core的个数,一个core同一时间只能运行一个task.
+ Worker : 集群中可以运行Application代码的节点。在Spark on Yarn模式中指的就是NodeManager节点
+ task: 在Executor进程中执行任务的工作单元，多个Task组成一个Stage,注意transformation(转换)操作的宽依赖划分stage
+ Job : 包含多个Task组成的并行计算，是由Action行为触发的 ,注意action 操作触发 job
+ stage:每个job会根据宽依赖划分成多个stage,每个stage由多个task去执行,具体task数量取决于分区数量,或者指定的shuffle并行度
+ DAGScheduler : 根据Job构建基于Stage的DAG，并提交Stage给TaskScheduler，其划分Stage的依据是RDD之间的依赖关系
+ TaskScheduler : 将TaskSet提交给Worker（集群）运行，每个Executor运行什么Task就是在此处分配的。

#### DAGScheduler:是high-level(顶层)调度器,它负责将代码解析成 生成job,将DAG划分成 Stage之后交给taskScheduler
#### TaskScheduler:是low-level(底层)调度器,负责提交stage(task set)任务的集合给集群,让集群调度他们,并且如果有任务失败重试,有任务执行慢,推测执行

## spark 基本运行流程
+ application中构建sparkcontext,sparkcontext向资源管理器(例如 yarn)注册并且申请executor资源
+ yarn 分配给executor资源,
+ sparkContext 的DAGScheduler生成job,根据代码构建DAG图,并且将DGA拆分成stage,每个stage对应一个taskSet(任务集合),之后将taskset发送给taskScheduler 任务调取器,executor向任务调取器,申请task执行.
+ taskScheduler 发送task给executor执行,sparkcontext发送相关代码给executor
+ executor运行完毕task释放资源,executor是独立的jvm进程,每个task是一个线程,executor间不会直接进行通讯,每个executor内的task可以共享内存和磁盘
## driver端和executor端进行的rpc通讯主要包括
+ driver发送广播变量
+ driver发送依赖包和代码到executor
+ executor汇报task执行情况
+ driver收集jvm执行信息,在ui界面上点击executor的时候  
**图解**   
![图片](/static/img/up-592c83053a0de10844974db433d34c2aa80.png)   
[参考连接](https://www.cnblogs.com/frankdeng/p/9301485.html)


## spark sql 的执行过程
### sql到rdd的过程要经过一个catalyst(催化器),他是spark sql的一个核心组件, 下面是详细过程
+ (解析)SQL 语句经过 SqlParser 解析成**未明确的逻辑计划** Unresolved LogicalPlan,是一个抽象语法树 AST;
+ (分析)使用 analyzer(分析器) 结合数据数据字典 (catalog) 进行绑定, 生成**明确的逻辑计划** resolved LogicalPlan;
+ (优化)使用 optimizer(优化器) 对 resolved LogicalPlan 进行优化,合并常量/join谓词下推/列裁剪, 生成 optimized LogicalPlan(已优化的逻辑计划);
+ (执行)使用 SparkPlan 将 LogicalPlan 转换成 PhysicalPlan(物理计划);
+ CostModel 根据过去的性能统计,选择最佳的物理执行计划
+ 使用 prepareForExecution() 将 PhysicalPlan 转换成可执行物理计划;
+ 使用 execute() 执行可执行物理计划;
+ 生成 RDD 之后就到DAG是如何调度的了
  ![图片](/static/img/sql.png)
参考链接 https://cloud.tencent.com/developer/article/2008340

#### 联系邮箱 xxx_xxx@aliyun.com