#  flink 的分区器
### [go back](/x2q/flink/flink)      
### [go home](/x2q)       
## 概述 
flink的分区器都继承自**StreamPartitioner**这个抽象类，StreamPartitioner实现自**ChannelSelector**接口，   
ChannelSelector：定义了三个方法 
+ setup：初始化 channel数量
+ selectChannel：选择哪个管道
+ isBroadcast：是否是广播模式
flink 一共有8个分区器，但是我们无法手动指定分区器，除了自定义的分区器，因为这些分区器的具体使用是根据选择的算子
自动选择的，例如：使用了keyby算子它的分区器就是KeyGroupStreamPartitioner，根据hash值进行分区
## 具体分区器如下
+ GlobalPartitioner：对每条记录，只选择下游operator的第一个Channel，算子global()
+ ShufflePartitioner：将记录随机输出到下游Operator的每个实例。算子shuffle()
+ RebalancePartitioner：将记录以循环的方式输出到下游Operator的每个实例。算子rebalance()
+ RescalePartitioner: 基于上下游Operator的并行度，将记录以循环的方式输出到下游Operator的每个实例。算子rescale()
+ BroadcastPartitioner：广播分区将上游数据集输出到下游Operator的每个实例中。算子broadcast()
+ ForwardPartitioner：将记录输出到下游本地的operator实例，区器要求上下游算子并行度一样，forward()
+ KeyGroupStreamPartitioner：将记录按Key的Hash值输出到下游Operator实例，算子keyBy()
+ CustomPartitionerWrapper:自定义分区器，partitionCustom(new CustomPartitioner(),0)

## 分区器继承图
![图片](/static/img/flin.png)  
#### 联系邮箱 xxx_xxx@aliyun.com