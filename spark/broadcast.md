# spark 广播变量和累加器
### [go back](/x2q/spark/spark)      
### [go home](/x2q)  
## 简单示例
                                  
```scala
package cn.x2q

import org.apache.spark.{SparkConf, SparkContext}

/**
 * @Auther: huowang
 * @Date: 14:55:53 2020/6/4
 * @DES:   广播变量和累加器
 * @Modified By:
 */

object BroadcastAndAccumulator {

  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setAppName("BroadcastAndAccumulator").setMaster("local")
    val sc = new SparkContext(sparkConf)
    //匹配 姓名和性别
    val map  = Map("Fred" -> "man", "Wilma" -> "woman")
    val bmap=sc.broadcast(map)
    // 一个累加器 用于计算总的记录数
    val acc=sc.longAccumulator;
    // 每个人的成绩表
    val initialScores = Array(("Fred","java", 88.0), ("Fred","database", 95.0), ("Fred","c", 91.0),
      ("Wilma","java", 93.0), ("Wilma","database", 95.0), ("Wilma","c", 98.0))
    val rdd=sc.parallelize(initialScores,2)
        .map(x=>{
          // 补齐每个人的性别
          acc.add(1)
          (x._1,x._2,x._3,bmap.value.get(x._1).get)
        })
    rdd.collect().foreach(println(_))
    println("acc.count:"+acc.count)
    // 运行结果
           //(Fred,java,88.0,man)
           //(Fred,database,95.0,man)
           //(Fred,c,91.0,man)
           //(Wilma,java,93.0,woman)
           //(Wilma,database,95.0,woman)
           //(Wilma,c,98.0,woman)
           //acc.count:6
  }
}

```                                  

## broadcast广播变量 内部方法说明
+ bmap.value:获取广播变量的值,这个值是广播的类型,如果是map,还需要在get一步
+ bmap.destroy():销毁广播变量,这个将会销毁广播变量的所有数据和元数据,一旦销毁无法恢复,并且这个方法会一直阻塞,直到销毁完全结束
+ bmap.unpersist(false/true)/不持久:异步/同步删除所有发送到executor的广播变量副本,可以传入布尔值,决定是否阻塞,默认false不阻塞,异步
这个方法将销毁广播变量在每个executor上的副本,如果之后使用广播变量需要重新发送    
通过上面可以看出 广播变量在driver端也没有更新的方法,但是可以销毁重发,     
**自定义一个支持更新的广播变量**
                                       
```scala
import java.io.{ObjectInputStream, ObjectOutputStream}
import org.apache.spark.broadcast.Broadcast
import org.apache.spark.streaming.StreamingContext
import scala.reflect.ClassTag

/**
 * 广播变量包装器
 * @param ssc
 * @param _v
 * @param
 * @tparam T
 */
case class BroadcastWrapper[T: ClassTag] (
                                          @transient private val ssc: StreamingContext,
                                          @transient private val _v: T) {
  // @transient 注解标识 该变量不会被持久化
  @transient private var v = ssc.sparkContext.broadcast(_v)

  def update(newValue: T, blocking: Boolean = false): Unit = {
    v.unpersist(blocking)
    v = ssc.sparkContext.broadcast(newValue)
  }

  def value: T = v.value


  private def writeObject(out: ObjectOutputStream): Unit = {
    out.writeObject(v)
  }

  private def readObject(in: ObjectInputStream): Unit = {
    v = in.readObject().asInstanceOf[Broadcast[T]]
  }
 // 使用实例
 //val testB = BroadcastWrapper[Map[String, String]](ssc,Map("a"->1))
  // testB.update(newMap,false)
}

```
                                  
## 广播大变量 GB级别
发送broadcast如果过大,采用Object2ObjectOpenHashMap(),结构代替hashmap,并且可以吧一个5G的大广播变量拆分成5个1G的对象,
发送完毕之后driver端主动GC清除内存中无用的对象,注意 广播变量不是每个task发一份,而是每个executor发一份,因为executor内部
task之间是可以同享内存和磁盘的

## 累加器 Accumulator
累加器会在各个task中累加,然后再driver端汇总,也就是说只有job执行结束了(action操作之后)才能拿到累加器的值,
而且要注意的是如果有多个 action操作而且中间没有用cache等阻断,那么累加器会被执行多次累加

#### 联系邮箱 xxx_xxx@aliyun.com