#  flink 基本算子
### [go back](/x2q/flink/flink)      
### [go home](/x2q)      

## flink的几种流 
+ DataStream[T](数据流): 一个单纯的里面数据是T类型的流,可接算子 map/filter/flatMap/keyBy 等
+ KeyedStream[T, K](键控流):执行完keyBy算子后的流,可接状态,可以接滚动聚合算子sum/min/max/minBy/maxBy/reduce(这些算子本质基于状态)

## flink 基本算子
### 转换算子 transformation
+ map
+ filter
+ flatmap
+ bykey
+ split 可以给一个流打上标签,之后调用select算子检出特定标签的流,已经废弃被**getSideOutput**取代
+ select 可以选择特定的split打的标签,检出新的流
+ connect 把两个流连接到一起合成一个新的流,但是这个新的流里面还是两个流,内部保持两个流各自的数据和格式不变,
相互独立,datastream->connectedStream
+ CoMap/CoFlatmap 就是执行connet后在执行map或者flatmap,datastream->connectedStream->dataStream
+ union 可以合并多条流,要求数据结构都一样,datastream->datastream
+ **flatMapWithState** 这个比较特殊,携带状态的flatmap,R是输出的类型,S是状态的类型,不需要定义输出的类型,因为承接上个
算子的输入的;fun:(T,Option[S])=>(TraversableOnce[R],Option[S]) 输入是(输入类型,状态),输出是(定义的输出类型,状态)
注意状态第一次进入到时候可能是没有,所以是option类型
#### connect 和 union 的区别  
connect 只能操作两个流,union 可以操作多个流
union要求类型必须一致,connect可以类型不一致,之后在用coMap整成一致
### 滚动聚合算子
+ sum
+ max
+ min
## flink 基本函数概念
+ richFunction 富函数类, 该类通常包含 context上下文, 生命周期open/close方法,可以实现更复杂的功能
+ 匿名函数
+ 自定义函数

## processFunctionApi
可以访问时间戳,水位线,以及注册定时事件
+ ProcessFunction
+ KeyedProcessFunction
+ CoProcessFunction
+ ProcessJoinFunction
+ BroadcastProcessFunction
+ KeyedBroadcastProcessFunction
+ ProcessWindowFunction
+ ProcessAllWindowFunction

## process 和 apply 区别
这两个东西很像 都是可以拿到一批数据，进行自定义的计算，但是有区别如下
+ apply 只能用在 window后面,apply里面需要自己定义类继承 RichWindowFunction/WindowFunction，可以拿到 上下文和window对象
+ process可以接在任何算子后面，需要自己定义类，根据不同情况继承processFunctionApi各种实现，可以拿到上下文等

## join算子
join算在可以根据一定条件合并两个流,常规用法如下
```scala
stream.join(otherStream)
    .where(<KeySelector>)
    .equalTo(<KeySelector>)
    .window(<WindowAssigner>)
    .apply(<JoinFunction>)

// 例子
  as
      .join(bs)
      .where(_._1)
      .equalTo(_._1)
      .window(TumblingEventTimeWindows.of(Time.seconds(3)))
      .apply{
        (t1 : (String,String, Long), t2 : (String,String, Long), out : Collector[(String,String, Long,Long)]) =>
          println("t1=>"+t1+",时间=>"+Utils.getStringDate3(t1._3))
          println("t2=>"+t2+",时间=>"+Utils.getStringDate3(t2._3))
            // 直接拼接到一起
            out.collect((t1._2,t2._2,t1._3,t2._3))
      }.name("joinedStreams")
      .map(x=>{
        println("结果输出=>"+x)
      })
```

+ **此外**join,可以接 滚动,滑动,session等窗口,此外还有一个 intervalJoin(间隔join)可以实现模糊匹配,
+ **此外**flink目前不支持 leftJoin/rightJoin,可以使用coGroup 协调划分来自己实现

## flink sql 的join语法
参考文档 https://nightlies.apache.org/flink/flink-docs-release-1.19/zh/docs/dev/table/sql/queries/joins/
+ 1 正常的内连接 左连接 右连接 全连接, 这个通过状态中保留数据来实现, 主要 如果有一侧新增了满足条件的数据, 历史上全部符合条件都会被join出来(只要状态没过期)
+ intervalJoin 允许一个between时间间隔的join

```sql
SELECT *
FROM Orders o, Shipments s
WHERE o.id = s.order_id
AND o.order_time BETWEEN s.ship_time - INTERVAL '4' HOUR AND s.ship_time
```

+ Temporal join ,状态表/维表join, 历史上有满足条件的不会被join出来 只会join当前的, 可以制定时间处理时间或event time
```sql
+ SELECT 
     order_id,
     price,
     orders.currency,
     conversion_rate,
     order_time
FROM orders
LEFT JOIN currency_rates FOR SYSTEM_TIME AS OF orders.order_time
ON orders.currency = currency_rates.currency;
```

#### 联系邮箱 xxx_xxx@aliyun.com