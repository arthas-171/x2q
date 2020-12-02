#  flink 和 spark streaming的反压机制
### [go back](/x2q/flink/flink)      
### [go home](/x2q)       

## 什么是反压
反压就是反对积压,那什么是积压!?,对于流式处理框架来说,一个job会分为由多个上下游组成的task,
去完成,当上游的task发送的数据太多,下游的task处理不过来时就是积压,task之间传递数据,就算是
号称逐条处理的flink,也不是一条一条发送的,而是一小批次一小批次的通过网络发送,中间最多会涉及三层逐级的
如下图:
![图片](/static/img/2019-09-25-004607.jpg)  
对于夸jvm(也就是夸机器的task之间传输数据来说)
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











#### 联系邮箱 xxx_xxx@aliyun.com