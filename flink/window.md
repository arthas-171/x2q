#  flink window 窗口
### [go back](/flink.md)      
### [go home](../README.md)     

## window概念  
### window 会将无限的流数据切割分发到有限的bucket(桶)中去做计算  
## window 分类
### 按time时间触发类型(time window),简写 .timeWindow()
+ 滑动:Sliding Windows,两个参数,(size,slide) 大小 滑步,可以重叠,时间对齐
+ 滚动:Tumbling Windows,一个参数size,等时间间隔切分,翻转滚动,时间对齐,窗口长度固定,没有重叠,每个数据仅会被划分到一个窗口中去
+ 会话:session Windows,两个数据之间间隔的时间大于某个值,划定为两个窗口,窗口大小不一定一样(时间不对齐),最后一个窗口永远不会触发
.window(EventTimeSessionWindows.withGap(Time.minute(10)))
### 按count条数/次数触发类型(count window) 简写 .countWindow()
+ 滑动 .countWindow(5),5个一滚
+ 滚动 .countWindow(5,2),5个一个窗口,隔2个划一下
### 全局窗口 Global Windows
+ Global Windows,全部数据按key划分到一个窗口里面,需要自定义触发器,不然不会触发计算
### Window()必须在keyby后使用,如果想要全局 直接windowAll()

## 窗口分配器 window assigner,Window api里面第一个参数是窗口分配器
### 窗口分配器的作用就是分发每条数据到不同的窗口
flink提供好的 window assigner 有四个
+ tumbling 滚动
+ sliding 滑动
+ session 会话
+ global 全局
## Window function 对窗口收集到的数据执行什么函数
### 增量聚合函数 incremental aggregate functions
+ sum
+ count
+ reduceFunction
+ AggregateFunction
保持一个简单的状态,来一条数据进行一次计算
### 全窗口函数 full Window functions
+ processWindowFunction
+ fold()
+ apply()
先把窗口里面所有的数据都收集起来,等到Window结束的时候再全部遍历一遍进行计算

## 可选 trigger() 触发器
## 可选 evitor() 移除器
定义特定的移除数据逻辑,可以窗口触发前 也可以触发后
## 可选 allowedLateness()延时处理器 
允许处理迟到的数据,窗口已经关闭迟到的数据,窗口计算不再等待了
## 可选 sideOutputLateData() 讲迟到的数据放入侧输出流,可以放入另外一个流去做其他统计处理
## 可选 getSideOutput() 获取侧流输出



#### 联系邮箱 xxx_xxx@aliyun.com