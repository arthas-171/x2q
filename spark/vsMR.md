# spark为什么比MapReduce快
### [go back](/spark.md)      
### [go home](../README.md)     
## spark为什么比MapReduce快

+ spark task启动时间快,因为spark采用线程池的方式启动task任务,而Hadoop每次都创建新的进程
+ spark只有在shuffle的时候才将数据写入磁盘,Hadoop MapReduce多个mp作业之间交换数据也依赖于磁盘,如果MapReduce 分片设计的比较好会减少数据在多个map之间传递的可能性,最好的情况是 尽量少的map 同时保证 map需要的数据不会在出现两个或者两个以上的block上
+ spark缓存机制比Hadoop的MapReduce更高效一些,因此spark适合反复迭代计算同一批数据
#### 联系邮箱 xxx_xxx@aliyun.com

