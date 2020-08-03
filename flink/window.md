#  flink window 窗口
### [go back](/flink.md)      
### [go home](../README.md)     

## window概念  
### window 会将无限的流数据切割分发到有限的bucket(桶)中去做计算  
## window 分类
### 按time时间触发类型(time window)
+ 滑动:Sliding Windows,两个参数,(size,slide) 大小 滑步,可以重叠,时间对齐
+ 滚动:Tumbling Windows,一个参数size,等时间间隔切分,翻转滚动,时间对齐,窗口长度固定,没有重叠,每个数据仅会被划分到一个窗口中去
+ 会话:session Windows,两个数据之间间隔的时间大于某个值,划定为两个窗口,窗口大小不一定一样(时间不对齐),最后一个窗口永远不会触发
### 按count条数/次数触发类型(count window)
+ 滑动
+ 滚动
### 全局窗口 Global Windows
+ Global Windows,全部数据按key划分到一个窗口里面,需要自定义触发器,不然不会触发计算


#### 联系邮箱 xxx_xxx@aliyun.com