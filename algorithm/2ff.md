# 2分法查找
### [go back](/algorithm.md)      
### [go home](../README.md)     
## 代码实现,
                                                
```scala

/**
 * @Auther: huowang
 * @Date: 14:49:32 2020/5/18
 * @DES:   查找
 * @Modified By:
 */

object Search {
  /**
   * 二分法查找
   * 查找 key 在 arr中的位置
   * arr是有序数组
   * @param arr
   * @param start
   * @param end
   * @param key
   */
  def halfSearch(arr:Array[Int],start:Int,end:Int,key:Int):Int={
    // 如果截止位置小于起始位置 不合法 返回-1
    if(start>end){
      return -1
    }
    // 找到中间的位置
    val mid=(end-start)/2+start
    // 如果恰好是 直接返回
    if(arr(mid).equals(key)){
      return mid
    }
    
    if(arr(mid)>key){
      // 如果大于 递归调用此方法 数组起始位置不变  截止位置变为mid(中间位置)
      halfSearch(arr,start,mid,key)
    }else{
      // 如果小于 递归调用此方法 数组起始位置变为mid(中间位置) 截止位置不变
      halfSearch(arr,mid,end,key)
    }
  }

  def main(args: Array[String]): Unit = {
      val arr=Array(0,1,3,4,5,6,8,9,11)
    val key=9
    println(halfSearch(arr,0,arr.length,key))

  }
}


```



#### 联系邮箱 xxx_xxx@aliyun.com