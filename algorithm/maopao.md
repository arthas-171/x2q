# 冒泡排序
### [go back](/x2q/algorithm/algorithm)      
### [go home](/x2q)       
## 代码实现,时间复杂度 n*n,n是数组长度
                                         
```scala
/**
 * @Auther: huowang
 * @Date: 16:32:44 2020/6/2
 * @DES: 排序算法实现
 *      冒泡
 *      鸡尾酒排序
 *      插入排序
 *      快速排序
 * @Modified By:
 */

object OrderBy {

  /**
   * 冒泡排序每次比较相邻的元素如果前者大于后者就调换位置 这样可以保证最后一个元素一定是最大的
   * 第二次循环逻辑和第一次一模一样
   * 循环数组长度次就行了
   * 所以复杂程度是 n^2
   *
   * @param arr
   * @return
   */
  def maopao(arr:Array[Int]):Array[Int]={
    var tmp:Int=0

    for(a <- 0 to arr.length-1){
         for(b <- 0 to arr.length-2){
              if(arr(b)>arr(b+1)){
                tmp=arr(b+1)
                arr(b+1)=arr(b)
                arr(b)=tmp
              }
         }
    }
     arr
  }

  def maopao2(arr:Array[Int]):Array[Int]={
    var tmp=0
    var c=0
    while (c<arr.length) {
      for(b <- 0 to arr.length-2){
        if(arr(b)>arr(b+1)){
          tmp=arr(b+1)
          arr(b+1)=arr(b)
          arr(b)=tmp
        }
      }
      c=c+1
    }
    arr
  }

  def main(args: Array[String]): Unit = {
    var arr=Array(1,8,3,7,9,2,6)
    arr=maopao2(arr)
    println(arr.mkString(","))
  }
}

```



#### 联系邮箱 xxx_xxx@aliyun.com