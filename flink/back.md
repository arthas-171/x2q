#  flink 和 spark streaming的反压机制
### [go back](/x2q/flink/flink)      
### [go home](/x2q)       

## 什么是反压
反压就是反对积压,那什么是积压!?,对于流式处理框架来说,一个job会分为由多个上下游组成的task,
去完成,当上游的task发送的数据太多,下游的task处理不过来时就是积压,task之间传递数据,就算是
号称逐条处理的flink,也不是一条一条发送的,而是一小批次一小批次的通过网络发送,中间最多会涉及三层逐级的
如下图:
![图片](/static/img/2019-09-25-004607.jpg)  
对于跨jvm(也就是跨机器的task之间传输数据来说)
+ taskManager内部有 **NetWork** Buffer负责缓存第一层
+ NetWork依赖**netty**进行通讯,netty有ChannelOutbound/ChannelInbound  Buffer 进行第二次缓存
+ netty最终还要通过**socket**进行真正的网络间传输,socket还有Send/Receive Buffer
如果是同一个taskManager内的task数据传递就少一层socket   
夸jvm的传输图
![图片](/static/img/2019-09-25-004831.jpg)  
同一jvm内部传输图
![图片](/static/img/2019-09-25-004846.jpg)  
## flink 1.5以前的反压
flink 1.5以前依赖于tcp协议(socket是对tcp协议的封装)自身携带的反压功能,逐层向上传递压力
###  tcp协议是如何实现反压的
tcp协议发送网络数据包会包含三个信息,
+ Sequence number 数据包编号
+ ACK number,保证传输的可靠性
+  Window Size:主要是通过这个来控制速率,下游接收端会通过window size告诉上游还能接受多少数据,如果满了,上游就不在放松,但是还会定期询问
![图片](/static/img/get10.png)  
tcp 传输的窗口是一个滑动窗口机制,如下图
![图片](/static/img/get11.png)  
## flink 1.5以后的反压依赖于 credit类似于tcp协议的反压的window
首先为啥子要改,因为1.5以前的依赖于原生协议的反压机制有两个缺陷
+ 第一,层级太多,导致反压流程过长,进而反应迟钝
+ 第二,我们要注意到 network是工作在taskManager上的而不是具体的task,所以如果taskManager中有一个task
触发了网络层的反压,会影响到其他没有压力的task也被迫降低传输速度
### flink 1.5以后依赖于 credit-base 授信 
首先上游task会像下游发送数据的时候携带一个backLog(期望发送的数据条数),下游task会返回一个credit
授信额度(允许发送数据的条数),上游task更加授信额度去组织发送数据,如果额度为0则不再发送,但是仍然
会定时发送侦测包,去侦测下游的额度有没有新的空余,这样在task层面逻辑上做了关于压力的沟通,不再强硬的
依赖底层网络的反压机制,如下图,task间逻辑上"直接"进行了沟通
![图片](/static/img/16d7e00f28bee9d4.png)  
## flink 端到端的反压
上面说的都是task间的反压,但是实际的job还有source和sink,会涉及到外部系统,对于source,如果是kafka
flink支持静态的设置读取的最大量,来进行静态压力控制,对于sink如果是kafka,支持动态感知到写入的压力
传递反压,如果是其他的第三方系统,可以就要自己去实现类似功能了
## spark streaming的反压机制
+ 1.5以前没有,只有通过静态的设置Receiver(接受者)最大接收值 spark.streaming.receiver.maxRate
对于从kafka读取的可以设置spark.streaming.kafka.maxRatePerPartition 参数来限制每次作业中每个 
Kafka 分区最多读取的记录条数
+ 1.5以后引入**RateController(速率控制器)**,这个东西通过processingDelay(处理延迟),schedulingDelay(调度延迟)
算出一个最大处理速率,规定每秒能处理的最大记录数,具体入下
### (1)
RateController继承自 StreamingListener，其监听所有作业的 onBatchCompleted 事件，并且基于
 processingDelay 、schedulingDelay 、当前 Batch 处理的记录条数以及处理完成事件来估算出一个速率；
 这个速率主要用于更新流每秒能够处理的最大记录的条数
### (2)
InputDStreams 内部的 RateController 里面会存下计算好的最大速率，这个速率会在处理完 
onBatchCompleted 事件之后将计算好的速率推送到 ReceiverSupervisorImpl，这样接收器就
知道下一步应该接收多少数据了
### (3)
如果用户配置了 spark.streaming.receiver.maxRate 或 spark.streaming.kafka.maxRatePerPartition，
那么最后到底接收多少数据取决于三者的最小值   
如下图  
![图片](/static/img/p.png) 
![图片](/static/img/get8.PNG) 
![图片](/static/img/get9.PNG) 
+ input rate：即从数据源中接收数据的速率。
+ scheduling delay：即一个batch在等待前一个batch处理完成前在队列中所等待的时间。
+ processing time：数据的处理时间，即从接收到数据开始并进行数据转换存储操作的整个过程所使用的的时间
+ total delay：即schedule delay + processing time所使用的的时间
+ Kafka 0.10 direct stream: 从kafka读取数据速率
## 如何启动spark streaming的反压
配置参数spark.streaming.backpressure.enabled 设置为 true ,默认是false
### 其他相关参数
+ spark.streaming.backpressure.initialRate： 
启用反压机制时每个接收器接收第一批数据的初始最大速率。默认值没有设置。
+ spark.streaming.backpressure.rateEstimator：
速率估算器类，默认值为 pid ，目前 Spark 只支持这个，大家可以根据自己的需要实现。
+ spark.streaming.backpressure.pid.proportional：
用于响应错误的权重（最后批次和当前批次之间的更改）。默认值为1，只能设置成非负值。weight for response to "error" (change between last batch and this batch)
+ spark.streaming.backpressure.pid.integral：
错误积累的响应权重，具有抑制作用（有效阻尼）。默认值为 0.2 ，只能设置成非负值。weight for the response to the accumulation of error. This has a dampening effect.
+ spark.streaming.backpressure.pid.derived：
对错误趋势的响应权重。 这可能会引起 batch size 的波动，可以帮助快速增加/减少容量。默认值为0，只能设置成非负值。weight for the response to the trend in error. This can cause arbitrary/noise-induced fluctuations in batch size, but can also help react quickly to increased/reduced capacity.
+ spark.streaming.backpressure.pid.minRate：
可以估算的最低费率是多少。默认值为 100，只能设置成非负值




#### 联系邮箱 xxx_xxx@aliyun.com