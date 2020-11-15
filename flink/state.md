#  flink 中的状态 
### [go back](/x2q/flink)      
### [go home](/x2q)     

## 状态 state
+ 状态可以理解为一种**缓存**
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
## 关于 可以托管的 operation state 算子状态,非键控状态
不是键控的状态使用,也就是说非健控的算子可以使用这些状态,状态可以理解为缓存,
通过 实现 CheckpointedFunction接口来实现,实现两个方法
+ snapshotState 每次拍快照的时候实现的逻辑
+ initializeState 初始化的时候实现的逻辑
官方实例,积攒一批数据写入一次存储/或者下游
                                                            
```scala
package cn.ubd.plugs.sink

import org.apache.flink.api.common.state.{ListState, ListStateDescriptor}
import org.apache.flink.api.common.typeinfo.{TypeHint, TypeInformation}
import org.apache.flink.runtime.state.{FunctionInitializationContext, FunctionSnapshotContext}
import org.apache.flink.streaming.api.checkpoint.CheckpointedFunction
import org.apache.flink.streaming.api.functions.sink.SinkFunction
import org.apache.flink.streaming.api.functions.sink.filesystem.BucketAssigner.Context

import scala.collection.mutable.ListBuffer

/**
 * @Auther: huowang
 * @Date: 19:32:03 2020/8/17
 * @DES: flink 官方的使用算子状态的例子
 * @Modified By:
 */
class BufferingSink (threshold: Int = 0) extends SinkFunction[(String, Int)] with CheckpointedFunction {

  //@transient 标识该变量不会被持久化 用于缓存使用 不能被序列化
  @transient
  private var checkpointedState: ListState[(String, Int)] = _

  // 一个缓存集合
  private val bufferedElements = ListBuffer[(String, Int)]()

  /**
   * 处理 每个写入 sink的元素的时候
   * @param value
   * @param context
   */
  def invoke(value: (String, Int), context: Context): Unit = {
    bufferedElements += value
    // 如果缓存的元素等于某个阈值 那么执行真正的写入 下游存储的操作
    // 例如这里可以积攒一批数据一起写入存储 减少与存储的交互次数
    // 如果存储支持事务性,那么可以赞一批数据作为一次事务写入存储
    // 由于flink 算子内部具有容错性,回复快照不会丢失数据
    // 可以通过此做到 恰好一次性
    if (bufferedElements.size == threshold) {
      for (element <- bufferedElements) {
        // send it to the sink
      }
      bufferedElements.clear()
    }
  }

  /**
   * 每次拍摄快照的时候执行
   * @param context
   */
  override def snapshotState(context: FunctionSnapshotContext): Unit = {
    //清空状态,并且将缓存集合中的值写入状态中
    checkpointedState.clear()
    for (element <- bufferedElements) {
      checkpointedState.add(element)
    }
  }

  /**
   * 初始化 方法
   * @param context
   */
  override def initializeState(context: FunctionInitializationContext): Unit = {
    // 初始化一个 list类型的状态描述器
    val descriptor = new ListStateDescriptor[(String, Int)](
      "buffered-elements",
      TypeInformation.of(new TypeHint[(String, Int)]() {})
    )
   // 初始化list类型的状态
    checkpointedState = context.getOperatorStateStore.getListState(descriptor)
    // 当从原来的快照中恢复时 执行,把状态中的每个元素写入缓存集合
    if(context.isRestored) {
      for(element <- checkpointedState.get()) {
        bufferedElements += element
      }
    }
  }
}

```                                                            
                                                            

## 键控状态/分区状态 keyed state
由输入数据流的key进行维护和访问,每个key维护一个状态实例,将相同key所有的数据分区到同一个算子任务中去
这个任务维护和处理这个key的状态
+ Value state: 只能存一个值
+ list state :可以存一个list,该list可以访问单个元素,可以追加,可以全部覆盖
+ map state :可以存一个map<key,value>
+ Reducing state and Aggregating state 聚合状态,这个状态和list state很像,只是可以用来做聚合操作,
.add的时候不是加入list而是**聚合入**list,注意reduce的输入值和输出值类型一致,aggregate可以不一致
此外 mapWithState()/flatMapWithState,是两个本来无状态算子的携带状态特例的一个简写方式
### 注意使用键控状态之前 必须先创建一个**状态描述器**StateDescriptor
+ ValueState<T> getState(ValueStateDescriptor<T>)
+ ReducingState<T> getReducingState(ReducingStateDescriptor<T>)
+ ListState<T> getListState(ListStateDescriptor<T>)
+ AggregatingState<IN, OUT> getAggregatingState(AggregatingStateDescriptor<IN, ACC, OUT>)
+ FoldingState<T, ACC> getFoldingState(FoldingStateDescriptor<T, ACC>)
+ MapState<UK, UV> getMapState(MapStateDescriptor<UK, UV>)
实例代码 我们需要在richFunction中通过获取运行时上下文this.getRuntimeContext.getState() 获取状态
,获取状态的时候需要定义状态描述器StateDescriptor,实例代码如下
                                                          
```scala
 var sum: ValueState[(Long, Long)] = this.getRuntimeContext.getState(
      new ValueStateDescriptor[(Long, Long)]("testState", classOf[(Long, Long)])
    )
```                                                          
                                                          
## 状态的生命周期 TTL time-to-live
创建状态时可以指定状态数据生存时间等实例如下,一般不设置也可以
                                                                    
```scala
import org.apache.flink.api.common.state.StateTtlConfig
import org.apache.flink.api.common.state.ValueStateDescriptor
import org.apache.flink.api.common.time.Time

val ttlConfig = StateTtlConfig
    .newBuilder(Time.seconds(1))
    .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
    .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
    .build
    
val stateDescriptor = new ValueStateDescriptor[String]("testState", classOf[String])
stateDescriptor.enableTimeToLive(ttlConfig)
```
                                                                     
更新类型配置何时刷新状态TTL（默认为OnCreateAndWrite）：
+ StateTtlConfig.UpdateType.OnCreateAndWrite -仅在创建和写访问权限时
+ StateTtlConfig.UpdateType.OnReadAndWrite -也具有读取权限
状态可见性用于配置是否清除尚未过期的默认值（默认情况下NeverReturnExpired）：
+ StateTtlConfig.StateVisibility.NeverReturnExpired -永不返回过期值
+ StateTtlConfig.StateVisibility.ReturnExpiredIfNotCleanedUp -如果仍然可用，则返回




## 状态后端 state backends
状态的存储,访问,维护,由一个可插入的组件决定,这个组件就叫状态后端,除了这个还负责checkpoint将检查点写入远程存储
### 可选的状态后端
+ memory state backend 本地状态存储在taskmanager的jvm的堆上,checkpoint存在jobmanager的内存的jvm堆上.延迟低但是不稳定
+ FS state backend, 本地状态存储在taskmanager的jvm堆上,checkpoint存在远程持久化的FileSystem上
+ RocksDB state backend,将所有状态序列化之后存入本地的RocksDB上
env.setStateBackend(new RocksDBStateBackend) //需要引入RocksDB的maven依赖   
默认checkpoint不开启   
env.enableCheckpointing(1000毫秒)

## 注意 **状态**和**局部变量**的区别
例如如果我在richMap的方法中声明一个局部变量,它可以实现和状态一样的效果,而且成本更低,并且还可以不受必须在bykey后使用键控状态  
的限制,其实二者的本质区别在于发生异常情况的容错性,状态可以被状态后端托管,flink的容错性和主动的checkpoint都会持久化状态,因此
托管于hdfs或者rockdb中的状态,在故障恢复后不会丢失这是和局部变量的一点区别
#### 联系邮箱 xxx_xxx@aliyun.com