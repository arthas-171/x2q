#  spark 基本原理及概念
### [go back](/x2q/spark/spark)      
### [go home](/x2q)

## spark 运行架构

spark的节点分为 driver(驱动节点)和executor(执行节点),基于yarn来提交spark job分为两种模式client和cluster,
两种模式去区别在于 client模式将会把driver程序运行在执行spark-submit的机器上,而cluster会把driver程序传输到集群中的一个节点去执行,
 client模式如果中断了spark-submit进程,job将会结束
  
这里要区分一下yarn的 resourceManager/NodeManager, 这两个概念和spark的driver和executor是独立的 , 
在yarn的nodeManager上spark不仅可以运行 driver而且可以运行executor

## spark rdd
rdd是spark的一个重要概念,全称 弹性分布式数据集,spark的任何操作都是基于这种数据结构,spark程序的基本流程都是 提取/创建rdd,转换rdd,
基于不同的rdd进行计算,rdd 上层还有封装的 dataframe和dataset ,
+ rdd:最底层的spark数据结构,没有schema(可以理解为数据表结构),使用spark core 的api操作,spark core是底层api使用需注意优化
+ dataframe:有schema,可以直接注册成 table,使用sql操作或者使用spark sql的api,dataframe的api是顶层api做了一些优化
+ dataset:dataset可以看成是dataframe的特例, dataframe中存储的是弱类型row(可以理解成sql里的行),dataset必须制定一个强类型例如User

Rdd 源码中有5个基本的方法或者叫5个基本属性 源码注释 each Rdd is characterized by five main properties
+ A list of partitions 一个分区的列表
+ A function for computing each split 一个用户计算每个分片的函数
+ A list of dependencies on other RDDs 一个记录依赖分区的列表
+ Optionally, a Partitioner for key-value RDDs (e.g. to say that the RDD is hash-partitioned) 可选的 对于 k-v类型的rdd,会有一个分区器 默认是hash分区器
+ Optionally, a list of preferred locations to compute each split on (e.g. block locations for an HDFS file) 一个记录每个分片首行位置的列表

### rdd 的 lineage(血缘)和checkpoint(检查点) 
rdd通过这两个机制来保证rdd容错,如果某一步的计算失败了,rdd通过以上两个机制来重算
+ lineage:是对rdd转换的一系列操作的记录,但是这个是一个粗粒度的记录,只会记录到块的级别,不会记录到行的级别,如果一行出错了整块都要重算,这样的好处是
记录的代价比较低,如果每行都记录那么代价太高了,如果某个块失败了,就会根据这个lineage进行重算,如果是一个窄依赖例如map操作,子rdd的某个块丢失了,需要重算
父rdd,父rdd的全部数据都是子rdd需要的因此重算没有冗余计算,如果是宽依赖例如 reducebykey,父rdd的分区分别给不同的子rdd,如果某一个子rdd数据丢失那么重算
也要重算父rdd的全部数据,虽然该子rdd只需要一部分数据
+ checkpoint:为了减少这种代价需要考虑使用checkpoint,尤其是在计算步骤非常多并且涉及到很多宽依赖的时候,但是
checkpoint本质上是将数据存储到磁盘,因为应该考虑只在关键位置点设置比较好,不然也很浪费,对应spark streaming这类的应用应当设置checkpoint,以便程序中断续传
不丢失数据,但是checkpoint并不能精确的做到完全不丢失,checkpoint和rdd.cache 不一样 cache程序结束或者意外终止后就丢失了,无论cache的级别是内存还是磁盘

## spark 常用transformation(转换)算子
###  单 value 类型 
#### 输入分区与输出分区 1对1
+ map:对当前rdd的每个元素应用一个函数之后返回一个新的rdd
+ flatMap:这个有点牛逼, 分两部分 第一步是对 对当前rdd的每个元素应用一个函数之后返回一个集合,然后第二部会把这集合转成rdd,
也就是说这个flatmap是可以一边转换数据结构一边做过滤操作的
+ mapPartitions:对当前rdd的每个分区应用一个函数返回一个rdd,这个也就是说会在每个task上应用一个函数,这个里面可以创建**连接**之类的操作,例如连接kakfa,hbase之类的
避免建立过多连接,它和foreachPartition()的区别是后者是一个action操作,没有返回值,前者是transformation操作返回一个rdd
+ glom: 将每个分区整成一个数组
#### 输入分区与输出分区 一对多
+ union 参考sql union, 连接两个集合
+ cartesian 对两个两个rdd内的元素进行求笛卡尔积的操作
#### 输入分区与输出分区 多对多
+ groupBy 只是根据某个属性进行分组
#### 输出分区为输入分区子集类型
+ filter 过滤
+ distinct 去重复
+ subtract 集合求差集
+ sample 相对比例采样
+ takeSample 设定个数采样
#### 缓存类型
+ cache 默认缓存到内存
+ persist 可以指定缓存级别
### key-value类型的transformation算子
#### 输入分区与输出分区一对一
+ mapValues 对key-value的数的 value进行map操作,这个和map的区别并且 mapvalues是不会改变rdd的分区器 
#### 对单个rdd进行聚合
+ combineByKey:这个可以精细化的聚合,有三个函数,f1:初始化数据类型,f2:单个元素往分区上合并,f3:分区间合并
+ reduceByKey:直接进行reduce操作输出结果和输入结果类型是一样的
+ foldBykey(v)(f):这个和reduceBykey一样,但是可以给一个初始值
+ partitionByKey:根据key重新分区
+ groupByKey:不会进行map端的预合并
+ aggregateByKey:给一个初始类型和初始值,之后两个函数,第一个函数是分区内合并,第二个是分区间合并
```java
   // reduceByKey()：没有初始值，分区内聚合和分区间聚合逻辑一样
   // foldByKey()： 有初始值，分区内聚合和分区间聚合逻辑一样
   // aggregateByKey()：有初始值，分区内和分区间逻辑可以不同
   // combineByKey() ：初始值可以变化结构，分区内和分区间逻辑不同
        
  //  combineByKey 和 aggregateByKey的区别
  //  虽然两个都是可以在分区里面先预聚合 但是 combinebykey 可以修改初始化的类型, 直接看源码
   def combineByKey[C](
           createCombiner: V => C,   ///  上游的是v这个类型,  下游的是c这个类型
           mergeValue: (C, V) => C,
           mergeCombiners: (C, C) => C,
           partitioner: Partitioner,
           mapSideCombine: Boolean = true,
           serializer: Serializer = null): RDD[(K, C)] = self.withScope {
           combineByKeyWithClassTag(createCombiner, mergeValue, mergeCombiners,
           partitioner, mapSideCombine, serializer)(null)
           }
```

### 两个rdd聚合
cogruop 对两个rdd进行协同划分
**例如**   
rdd1:[(1,a),(2,b),(3,c)]  
rdd2:[(1,10),(1,11),(2,20),(3,30),(3,33)]   
结果
[(1,(a,[10,11])),(2,(b,[20])),(3,(c,[30,33]))] 还是'三行'数据
### 连接
+ jion 参考sql连接操作
+ leftOutJoin/rightOutJoin 参考sql连接操作
### 无输出类型
+ foreach(f) 对每个集合元素应用函数f
+ foreachPartition() 对每个分区上应用函数
## spark action(行动)算子
+ saveAsTextFile算子 输出到hdfs指定目录
+ saveAsObjectFile算子 将分区中的每10个元素组成一个Array，然后将这个Array序列化，映射为（Null，BytesWritable（Y））的元素，写入HDFS为SequenceFile的格式




#### 联系邮箱 xxx_xxx@aliyun.com