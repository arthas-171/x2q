# spark 重新分区 repartition和coalesce 
### [go back](/spark.md)      
### [go home](../README.md)     
## spark 重新分区 repartition和coalesce
### spark 重新分区的优势
+ 对于给定RDD只需要扫描一次的情况 重新分区没有任何好处
+ 类似于 join() groupbykey() reducebykey() 这样的操作都会受
  
reparation是coalesce的特殊情况 ,reparation会将coalesce中的shuffle参数设置为true,会重新混洗分区,如果原有分区数据不均匀可以用reparation来重新混洗分区,使数据均匀分布,重新混洗过的分区和新的分区时宽依赖的关系
 
coalesce shuffle参数为false的情况 不会重新混洗分区,它是合并分区,比如把原来1000个分区合并成100个,父rdd和子rdd是窄依赖

在程序要输出结果的时候,可以通过控制分区来控制每个输出文件的大小,使文件大小合理,可以通过计算总条数/每个文件条数来计算,期望的分区数

**注意** 如果用coalesce从少分区扩展为多分区例如 100个分区到1000个分区,必须设置 参数为true 否则不会真正执行
#### 联系邮箱 xxx_xxx@aliyun.com

