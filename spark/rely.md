# spark的宽依赖和窄依赖
### [go back](/x2q/spark/spark)      
### [go home](/x2q)        
## spark的宽依赖和窄依赖

spark 划分stage取决于rdd之间的依赖,rdd之间的依赖分为宽依赖和窄依赖

+ 窄依赖是指 父rdd的一个分区指被子rdd的一个分区使用,父rdd只会被一个子rdd使用
+ 宽依赖父rdd的每一个分区都有可能被子rdd的分区使用,子rdd的分区通常对应父rdd的所有分区(partition)父rdd会被多个子rdd使用

**总结**
宽依赖一般是涉及到shuffle操作,窄依赖一般是map 这种操作, 宽依赖在丢失分区时可能会涉及到所有计算都要重新进行

常见窄依赖算子:map, filter, union, join(父RDD是hash-partitioned ), mapPartitions, mapValues

常见宽依赖算子:groupByKey, join(父RDD不是hash-partitioned ), partitionBy


#### 联系邮箱 xxx_xxx@aliyun.com