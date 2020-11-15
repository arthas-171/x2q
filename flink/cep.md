#  flink CEP 复杂事件处理 complex Event Processing
### [go back](/flink.md)      
### [go home](../README.md)     

## cep 什么是复杂事件处理
复杂事件就是**连续到来的若干条数据符合一定的规律**,例如 某个数据流的数据有 a,b,c,d 四种类型,他们是随机到来的
但是我们想要检测,a后面紧接着c的这种情况,数据是否在时间是乱序,是water mark负责的问题,cep不用关心   
a,c,d,d,d,a,a,a,b,b,c,c,d,d,a,b,c,d,a,c,b,c,d->ac,ac
其实这个东西我们通过 状态+定时器完全可以实现,但是为了编程的简单可以选择使用CEP,CEP的底层实现也是基于状态和定时器的
## 如何使用CEP
+ 先定义一个 CEP
+ 再把这个定义好的CEP 应用到一个流上
+ 再定义一个检出器,应用检出器检出数据
## 定义 CEP
实例代码
                            
```scala
 val loginFailPattern = Pattern.begin[LoginEvent]("begin").where(_.eventType == "fail")
      .next("next").where(_.eventType == "fail")
      .within(Time.seconds(3))
```                            
                            
+ begin:必须 begin开始,名字"begin"是随便起的,但是检出的时候会根据这个名字进行检出
+ where: 里面跟入的是具体的 表达式逻辑
+ next: 表示**紧随**,例如 a,b,c里面ab就是紧随的.ac就不是紧随的
+ followedBy:表示**跟随**,只要出现在后面就行,例如 a,b,c=>ac就是跟随
+ followedByAny:表示**非确定性跟随**.例如a,b,c1,c2,=>ac1,ac2
+ notNext:不紧随
+ notFollowedBy:不是不跟随,是不让某个事件在两个事件之间
                           
### 量词
+ start.times(3) 匹配三次
+ start.times(1,3).greedy 匹配出现 1次 或者 2次 或者 3次 并且尽可能多的重复匹配  **greedy贪婪尽可能多匹配**
+ start.times(1,3) 匹配出现 1次或者2次或者3次
+ start.times(3).optional 匹配出现 0次或者4次 **optional可以有可以无**
+ start.timesOrMore(2).optional.greedy 0次或者2次或者多次,并且尽可能多的重复匹配    
+ start.oneOreMore 匹配1次或者多次
### 条件符
+ where:多个where之间是and
+ or:or相当于或者的where
+ until:终止条件如果前面有了 oneOrMore或者oneOrMore.optional,建议必须使用      
+ 迭代条件 where((value,ctx)=>{}) 可以获取上下文            
+ 时间限制:within(Time.second(3)) 3秒内匹配有效
### 注意
+ 必须用begin开头
+ 不能以notFollowedBy结束
+ not类型的不能用 **optional可有可无** 修饰
+ 最好指定事件约束.within(Time.second(3)) 三秒内匹配有效
## 定义检出器
                               
```scala
class TestMatch() extends PatternSelectFunction[Xl, String]{
  override def select(map: util.Map[String, util.List[Xl]]): String = {
    val first = map.get("begin").iterator().next()
    val last = map.get("next").iterator().next()
    first.deviceNumber.concat("开始时间").concat(first.startTime).concat("结束时间").concat(last.startTime)
  }
}
```                               
                               
map的key是前面操作符的名字,继承PatternSelectFunction重写就行,因为是一个无限的流所以拿到的是一个迭代器,每次取一个匹配到
的值,当然理论上你也可以转成list全部取出,输出结果是自定义的
## 完整示例
                                          
```scala
package cn.ubd.realtime.example

import java.util

import akka.event.Logging.Warning
import cn.ubd.plugs.model.Xl
import cn.ubd.plugs.source.ReadFileSource
import cn.ubd.plugs.utils.UbdGeohash
import org.apache.commons.lang3.time.FastDateFormat
import org.apache.flink.cep.PatternSelectFunction
import org.apache.flink.cep.scala.CEP
import org.apache.flink.cep.scala.pattern.Pattern
import org.apache.flink.streaming.api.TimeCharacteristic
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor
import org.apache.flink.streaming.api.scala.{DataStream, StreamExecutionEnvironment}
import org.apache.flink.streaming.api.windowing.time.Time

/**
 * @Auther: huowang
 * @Date: 16:33:12 2020/8/30
 * @DES:  cep 例子
 * @Modified By:
 */
object TestCEP {

  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
    val dateFmt = "yyyy-MM-dd HH:mm:ss" // 时间格式
    val fdf=FastDateFormat.getInstance(dateFmt);
    import org.apache.flink.api.scala._
    val dataStream:DataStream[String] = env.addSource(new ReadFileSource("D:\\unicom\\workspace\\DataGovernance\\ln\\UbdXlRealtime\\UbdGeofence\\data\\xl.log"))
   val byKeyStream= dataStream.flatMap(x=>{
      val arr= x.split("\\|")
      // 过滤不合规数据
      if(arr.length==23 && !arr(0).isEmpty && ( !arr(2).isEmpty || !arr(3).isEmpty)){
        // 信令发生时间戳,取信令开始时间,如果没有就取截止时间
        var startTs:Long=0L
        var startTime:String=""
        if(!arr(2).isEmpty){
          startTime=arr(2).trim
          startTs= fdf.parse(arr(2)).getTime
        }else{
          startTime=arr(3).trim
          startTs= fdf.parse(arr(3)).getTime
        }
        // 如果 经度是空值 则支撑 0,0
        if(null==arr(16) || arr(16).trim.isEmpty){
          arr(16)="0"
        }
        // 纬度
        if(null==arr(15) || arr(15).trim.isEmpty){
          arr(15)="0"
        }
        Some(Xl(arr(0).trim,
          arr(1).trim,
          startTime,
          startTs,
          arr(4).trim,
          arr(5).trim,
          arr(6).trim,
          arr(8).trim,
          arr(9).trim,
          arr(10).trim,
          arr(11).trim,
          arr(12).trim,
          arr(16).trim,//经度
          arr(15).trim,//纬度
          arr(19).trim,
          "geoHash"
        ))
      }else{
        None
      }
    }).assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor[Xl](Time.seconds(5)) {
      override def extractTimestamp(element: Xl): Long = element.startTs
    })
      .keyBy(_.deviceNumber)
    // 匹配 三秒内 事件类型都是 43的数据
    val testPattern = Pattern.begin[Xl]("begin").where(_.eventType == "43")
      .next("next").where(_.eventType == "43")
      .within(Time.seconds(3))

    val patternStream = CEP.pattern(byKeyStream, testPattern)
    patternStream.select(new TestMatch).print("结束输出")
    env.execute("TestCEP")
  }

}

/**
 * 检出方法
 */
class TestMatch() extends PatternSelectFunction[Xl, String]{
  override def select(map: util.Map[String, util.List[Xl]]): String = {
    val first = map.get("begin").iterator().next()
    val last = map.get("next").iterator().next()
    first.deviceNumber.concat("开始时间").concat(first.startTime).concat("结束时间").concat(last.startTime)
  }
}
```                                          
                                          
                                                                     



#### 联系邮箱 xxx_xxx@aliyun.com