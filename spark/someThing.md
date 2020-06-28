# spark变成技巧
### [go back](/spark.md)      
### [go home](../README.md)     
## spark变成技巧
### 使用更高效的数据结构(完全用基础类型可能会带来代码的难以阅读性,增加维护难度,多加点注释就行了)
+ 使用固定分隔符的string代替自定义对象
+ 使用数组代替其他数据结构,或者 Arraybuffer
+ 发送broadcast如果过大,采用Object2ObjectOpenHashMap(),结构代替hashmap,并且可以吧一个5G的大广播变量拆分成5个1G的对象,
发送完毕之后driver端主动GC清除内存中无用的对象
### 使用高效的算子
+ 能用reduceBykey 就用reduceBykey
+ 不行就用combineBykey
+ union+reduceBykey替代一些join操作
+ 利用cache/persist 缓存重复计算
+ mappartition/foreachPartition中创建连接
### 输出文件大小控制
+ 例如coalesce+数据总数/每个文件行数,控制输出文件的大小,不要输出过多小文件
+ 直接获取rdd的分区数,在进行合并分区也可以控制小文件,不用计算数据总条数
+ 对于kafka可以利用原始api获取到 offset,从而计算消费的数据量
#### 联系邮箱 xxx_xxx@aliyun.com

