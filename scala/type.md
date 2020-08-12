Scala中的数据类型
### [go back](/scala.md)      
### [go home](../README.md)     
  
## double 和 float 不能用equal比较大小 ,应该用==
                                                
```scala
object TestDouble {

  def main(args: Array[String]): Unit = {
   val d:Double=0.0
    val f:Float=0
    if(d.equals(0)){
      println("equal")
    }
    if(d==0){
      println("==")
    }
    if(f.equals(0)){
      println("fequal")
    }
    if(f==0){
      println("f==")
    }
  }
//  ==
//  f==
}
```                                                
                                                








#### 联系邮箱 xxx_xxx@aliyun.com
