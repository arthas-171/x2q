#  flink 时间语义和水位线
### [go back](/x2q/flink/flink)      
### [go home](/x2q)       

## flink 时间语义
### Event time,事件创建时间
+ env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime) 告诉flink要使用何种时间语义
                                                  
````scala
    WatermarkStrategy
      .forBoundedOutOfOrderness[(Long, String)](Duration.ofSeconds(20)) //指定水位线为延迟 20秒
      .withTimestampAssigner(new SerializableTimestampAssigner[(Long, String)] {
        // 提取数据中 第一个long型字段为eventTime 注意这个数据必须是一个毫秒值时间戳
        override def extractTimestamp(element: (Long, String), recordTimestamp: Long): Long = element._1  
      })
````                                                  
                                                  
+ source.assignTimestampsAndWatermarks(WatermarkStrategy)                                                  
### Ingestion time,事件进入flink时间
### Processing time,事件在operation算子处理时的时间,当时运行时机器的时间

## watermark 水位线/水印
就相当于在通勤车的司机,明明到了发车的时间了,但是看车内人很少就在多等几分钟,允许迟到的人也赶上这趟班车  
其实就是允许一定时间内迟到的数据参与该窗口的计算,窗口的计算触发时间也会延迟.
+ watermark 是衡量eventTime进展的机制,可以设置一个延迟
+ watermark 通常用来配合Window处理乱序的数据,等待一会,等乱序的数据都到了(部分到了),重新排序一起处理,也可以不排序
根据业务而定
+ 数据流中的watermark 用于表示timestamp小于watermark的数据都已经到达了,因此Window计算的触发也是由watermark触发的
例如 应该16:05:00 触发窗口计算,其实是watermark水位线高于16:05:00了
+ watermark 让程序编程者自己去权衡数据的延迟和乱序数据的正确性,如果你把watermark设置成延迟一天那么乱序数据
应该肯定是能够按顺序处理,但是延迟太高了
## watermark的实现机理
+ watermark 可以认为是一条插入到数据量中的特殊的时间戳数据
+ watermark 是单向递增的
+ watermark 与数据的时间戳相关
![图片](/static/img/20200804161139.png)  
## watermark 任务之间的传递
每个task有一个自身的task clock任务时钟,task会取各个分区里面最小的一个watermark最为它的watermark传递给下游的task
一个分区的watermark来了时候会与该分区的watermark比较 大于则更新,之后task会比较全部分区的watermark,如果选取最小的
watermark作为task的watermark,可能新来的并不是最小的   
多个输入流的话以最小的watermark为准触发计算
![图片](/static/img/20200804163136.png)  

## 水印生成策略
+ 周期性生成水印,常用的BoundedOutOfOrdernessGenerator是定期水印生成
+ 特殊标识水印生成
### 特殊标识水印生成(标点符号)水印生成器
这种水印生成器,可以允许我们自己定义生成策略,根据某些特殊的数据的到来 生成水印   
理论上可以给每条数据都打伤水印,但是**注意**!每个水印都会在下游触发一系列操作,
因此flink不建议打过多的水印,这将降低flink的处理效率   
表单符号水印实例如下:
                                                         
                                                         
```scala
class PunctuatedAssigner extends AssignerWithPunctuatedWatermarks[MyEvent] {

    override def onEvent(element: MyEvent, eventTimestamp: Long): Unit = {
        if (event.hasWatermarkMarker()) {
            output.emitWatermark(new Watermark(event.getWatermarkTimestamp()))
        }
    }

    override def onPeriodicEmit(): Unit = {
        // don't need to do anything because we emit in reaction to events above
    }
}
```                                                         
#### 联系邮箱 xxx_xxx@aliyun.com