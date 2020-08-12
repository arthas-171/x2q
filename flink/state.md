#  flink 中的状态 
### [go back](/flink.md)      
### [go home](../README.md)     

## 状态 state
+ 由任务维护用于计算某个结果的所有数据,都属于这个任务的**状态**
+ 可以认为状态就是一个本地变量,可以被任务的业务逻辑访问
+ flink会对**状态**进行管理,包括**状态一致性**,**故障处理**和**高效存储于访问**,让开发人员只专注于逻辑
![图片](/static/img/get1.PNG)  


## 算子状态 operation state
作用在算子任务内,算子对应多个子任务task,每个子任务task有自己的状态,算子之间不能访问,同一个算子内子任务之间也不能互相访问
状态是对应task的
+ list state 列表状态,将状态表示为一组数据的列表,
+ union list state 联合列表状态,和列表状态基本一样,就是在故障恢复的时候不一样
+ broadcast 广播状态会把同一份状态广播给每一个算子子任务
## 键控状态/分区状态 keyed state
由输入数据流的key进行维护和访问,每个key维护一个状态实例,将相同key所有的数据分区到同一个算子任务中去
这个任务维护和处理这个key的状态
+ Value state 值装填
+ list state 列表状态 将状态表示为一组列表,可以用来存之前的一组数据
+ map state 映射状态 将状态表示为 k-v
+ Reducing state and Aggregating state 聚合状态,这个状态和list state很像,只是可以用来做聚合操作,
.add的时候不是加入list而是**聚合入**list

## 状态后端 state backends
状态的存储,访问,维护,由一个可插入的组件决定,这个组件就叫状态后端,除了这个还负责checkpoint将检查点写入远程存储
### 可选的状态后端
+ memory state backend 本地状态存储在taskmanager的jvm的堆上,checkpoint存在jobmanager的内存的jvm堆上.延迟低但是不稳定
+ FS state backend, 本地状态存储在taskmanager的jvm堆上,checkpoint存在远程持久化的FileSystem上
+ RocksDB state backend,将所有状态序列化之后存入本地的RocksDB上
env.setStateBackend(new RocksDBStateBackend) //需要引入RocksDB的maven依赖   
默认checkpoint不开启   
env.enableCheckpointing(1000毫秒)

#### 联系邮箱 xxx_xxx@aliyun.com