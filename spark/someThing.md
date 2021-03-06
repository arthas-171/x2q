# spark变成技巧
### [go back](/x2q/spark/spark)      
### [go home](/x2q)         
## spark编程技巧
### 使用更高效的数据结构(完全用基础类型可能会带来代码的难以阅读性,增加维护难度,多加点注释就行了)
+ 使用固定分隔符的string代替自定义对象
+ 使用数组代替其他数据结构,或者 Arraybuffer  
### 使用高效的算子
+ 能用reduceBykey 就用reduceBykey
+ 不行就用combineBykey
+ union+reduceBykey替代一些join操作
+ 利用cache/persist 缓存重复计算
+ mappartition/foreachPartition中创建连接
+ flatMap 替换 map+filter 减少一次数据遍历
#### flatMap 示例代码
                                                                           
```scala
package cn.x2q

import org.apache.spark.{SparkConf, SparkContext}

/**
 * @Auther: huowang
 * @Date: 14:55:53 2020/7/21
 * @DES:   测试算子 flatMap
 * @Modified By:
 */

object TestFlatMap {

  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setAppName("TestFlatMap").setMaster("local")
    val sc = new SparkContext(sparkConf)
    val initialScores = Array(("Fred", 88.0), ("Fred", 95.0), ("Fred", 91.0), ("Wilma", 93.0), ("Wilma", 95.0), ("Wilma", 98.0))
    val rdd=sc.parallelize(initialScores,2)
      .flatMap(x=>{
        // 过滤90分以上的人 并且打上标记合格
        if(x._2>=90){
          Some(x._1+"合格",x._2)
        }else{
          None
        }
      }).collect().foreach(println(_))

//    结果
//    (Fred合格,95.0)
//    (Fred合格,91.0)
//    (Wilma合格,93.0)
//    (Wilma合格,95.0)
//    (Wilma合格,98.0)


  }

}

```                                                                           
                                                                           
### 输出文件大小控制
+ 例如coalesce+数据总数/每个文件行数,控制输出文件的大小,不要输出过多小文件
+ 直接获取rdd的分区数,在进行合并分区也可以控制小文件,不用计算数据总条数
+ 对于kafka可以利用原始api获取到 offset,从而计算消费的数据量

### 读取hdfs文件,合并小文件防止初始化task过多
直接使用 sc.textFile("hdfs:///dir/")的方式读取hdfs上的文件,会为每个文件生成一个task,这样如果小文件特别多,初始化的task就会特别多
但是每个task处理的数据量非常小,这样不好
                                               
                                                
使用  CombineTextInputFormat,配合hadoopConfiguration 配置 可以合并小文件为 split ,每个split一个task                   
```scala

  val hadoopConfiguration = sc.hadoopConfiguration
    hadoopConfiguration.set("mapreduce.input.fileinputformat.split.maxsize", props.getProperty("mapreduce.input.fileinputformat.split.maxsize"))
    hadoopConfiguration.set("mapreduce.input.fileinputformat.split.minsize", props.getProperty("mapreduce.input.fileinputformat.split.minsize"))
    hadoopConfiguration.set("mapreduce.fileoutputcommitter.marksuccessfuljobs", props.getProperty("mapreduce.fileoutputcommitter.marksuccessfuljobs"))
    hadoopConfiguration.set("mapreduce.input.fileinputformat.split.minsize.per.node", props.getProperty("mapreduce.input.fileinputformat.split.minsize.per.node"))
    hadoopConfiguration.set("mapreduce.input.fileinputformat.split.minsize.per.rack", props.getProperty("mapreduce.input.fileinputformat.split.minsize.per.rack"))
    hadoopConfiguration

 val rdd= session.sparkContext.newAPIHadoopFile("inputPath",
      classOf[CombineTextInputFormat],
      classOf[LongWritable], classOf[Text],
      confhadoopConfiguration

```
                                        

#### 联系邮箱 xxx_xxx@aliyun.com

