# spark 重新分区 repartition和coalesce & 分区器
### [go back](/x2q/spark/spark)      
### [go home](/x2q)     
## spark 重新分区 repartition和coalesce
### spark 重新分区的优势
+ 对于给定RDD只需要扫描一次的情况 重新分区没有任何好处
+ 类似于 join() groupbykey() reducebykey() 这样的操作都会受
  
reparation是coalesce的特殊情况 ,reparation会将coalesce中的shuffle参数设置为true,会重新混洗分区,如果原有分区数据不均匀可以用reparation来重新混洗分区,使数据均匀分布,重新混洗过的分区和新的分区时宽依赖的关系
 
coalesce shuffle参数为false的情况 不会重新混洗分区,它是合并分区,比如把原来1000个分区合并成100个,父rdd和子rdd是窄依赖

在程序要输出结果的时候,可以通过控制分区来控制每个输出文件的大小,使文件大小合理,可以通过计算总条数/每个文件条数来计算,期望的分区数

**注意** 如果用coalesce从少分区扩展为多分区例如 100个分区到1000个分区,必须设置 参数为true 否则不会真正执行

### spark 里面的分区器
Hash 分区为当前的默认分区。 分区器直接决定了 RDD 中分区的个数、RDD 中每条数据经过 Shuffle后进入哪个分区和 Reduce 的个数。 
+ （1）只有Key-Value类型的pairRDD才有分区器，非Key-Value类型的RDD分区的值是None
+ （2）每个RDD的分区ID范围：0~numPartitions-1，决定这个值是属于那个分区的。
+ rdd.partitioner：查看对应分区器
+ partitionBy()：对 pairRDD 进行分区操作，如果原有的 partitionRDD 的分区器和传入的分区器相同,
则返回原 pairRDD，否则会生成 ShuffleRDD，即会产生 shuffle 过程
spark中有3个分区器
#### 默认的HashPartitioner
#### 排序的时候用的分区器 RangePartitioner
范围分区器，排序内部默认使用这个分区器。将一定范围内的数映射到某一个分区内，尽量保证每个分区中数据量的均匀
，而且分区与分区之间是有序的，一个分区中的元素肯定都是比另一个分区内的元素小或者大，但是分区内的元素是不能保证顺序的。
原理：通过水塘抽样算法确定边界数组，再根据key来获取所在的分区索引
#### 自定义分区器
```commandline
//自定义的Hash分区器
class CustomPartitioner(numPar: Int) extends Partitioner {
  assert(numPar > 0)
  
  // 返回分区数, 必须要大于0.
  override def numPartitions: Int = numPar

  //返回指定键的分区编号(0到numPartitions-1)
  override def getPartition(key: Any): Int = key match {
    case null => 0
    case _ => key.hashCode().abs % numPar
  }
}

object MyCustomPartitioner {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf().setAppName("MyCustomPartitioner").setMaster("local[2]")
    val sc: SparkContext = new SparkContext(conf)
    sc.setLogLevel("WARN")

    val rdd1 = sc.parallelize(Array((10, "a"), (20, "b"), (30, "c"), (40, "d"), (50, "e"), (60, "f")))
    println(rdd1.partitioner)

    //使用HashPartitioner重新分区
    val rdd2 = rdd1.partitionBy(new CustomPartitioner(2))
    println(rdd2.partitioner)

    //所有的数据都在0分区，因为key都是偶数
    rdd2.glom().map(_.toList).collect().foreach(println)
  }
}
```
+ Q map阶段如何确定分区数
  + Spark从HDFS读入文件的分区数默认等于HDFS文件的块数(blocks),如果一个文件大小超过了blocks的大小(一般默认128MB),那么会被多个任务执行
;如果小文件太多, 每个文件的大小都小于块的大小,例如只有几MB,这个时候每个文件会生成一个task,我们需要对读取小文件进行优化, 在读取的时候进行合并
;代码使用CombineTextInputFormat,sql可以使用配置, 针对perquet和orc可以分别设置读取时合并文件的大小, 还有可以开启小文件先调度的配置
+ Spark 如何设置合理分区数？
  + 总核数=executor-cores * num-executor 一般合理的分区数设置为总核数的2~3倍
  + 
  + 分区数太多意味着任务数太多，每次调度任务也是很耗时的，所以分区数太多会导致总体耗时增多。
  + 分区数太少的话，会导致一些结点没有分配到任务；另一方面，分区数少则每个分区要处理的数据量就会增大，从而对每个结点的内存要求就会提高
  分区数不合理，会导致数据倾斜问题





#### 联系邮箱 xxx_xxx@aliyun.com

