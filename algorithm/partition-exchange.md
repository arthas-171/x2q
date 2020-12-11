# 快速排序发（分区交换排序法）
### [go back](/x2q/algorithm/algorithm)      
### [go home](/x2q)    

## 概述 
快速排序算法又叫做分区交换排序法，显然后者称呼的更准确，最早是由东尼.霍尔提出的，最好的时间复杂度是 O(n*logn)，最坏事O(n*n)
## 基本思想
+ 选基:从结合中选出一个随机元素作为**基准**，pivot
+ 分割:把小于基准的元素，都放到基准前面，大于基准的元素放到基准后面，等于的可以放到任意一边，如此把集合切割成两个集合
+ 递归子集合:对两个子集合分别递归进行上述操作，直到得到的子集合长度为1，左右下标重合
## 代码实现
注意，代码实现的时候比较优秀的一种方式是采用**in-place（原地分隔）**，通过交换左右下标的位置
                                                               
                                                              
```scala
package cn.x2q.algorithm

/**
  * @Auther: huowang
  * @Date: 19:34:47 2020/12/10
  * @DES:  分区交换算法（快速排序发）
  * @Modified By:
  */
object PartitionExchange {

  /**
    * 切割分区,返回基准元素位置
    * @param arr
    * @param left
    * @param right
    * @return
    */
  def partition(arr:Array[Int],left:Int,right: Int):Int={
    // 获取基准元素 直接选取最右侧一个元素为基准元素,
    // 把小于等于该元素的元素都放到它的右边,大于的放到他左边
    val pv=arr(right)
    // 把最左边一个索引作为堆叠索引
    var storeIndex=left
    //操作数组-1 是因为最右边一个元素是基准元素
    for (i <- left to right-1 ){
       if(arr(i)<=pv){
         //把小于基准元素的元素 都堆到集合左端
         swap(arr,storeIndex,i)
         // 把用于堆叠索引往前移动一个
         storeIndex=storeIndex+1
       }
       //如果出现了比基准元素大的元素,那么则不会移动堆叠索引
       //但是如果之后又出现了比基准元素小的元素,那边会与这个大的元素交换位置
       //之后通过交换基准元素与堆叠索引的位置
       //进而使大的元素永远出现在堆叠索引右侧
    }
    // 这里最右边的元素,其实是基准元素,我们把基准元素和最后堆叠索引对应的元素调换位置
    // 这样基准元素右边就都是大于它的元素了
    swap(arr,right,storeIndex)
    // 返回堆叠索引位置,目前堆叠索引指向的就是基准元素
    storeIndex
  }

  def quicksort(arr:Array[Int],left: Int,right: Int):Array[Int]={

    if(right>left){
      // 左右索引不重合
      // 随便选择一个元素作为基准 就选择最左边的吧
      // 切割返回基准元素
      val pivotIndex= partition(arr,left,right)
      // 递归对切割形成的两个子集进行排序
      quicksort(arr,left,pivotIndex-1)
      quicksort(arr,pivotIndex,right)
    }
    arr
  }


  /**
    * 调换 a b 元素在数组中的位置
    * @param arr
    * @param a
    * @param b
    */
  def swap(arr:Array[Int],a:Int,b:Int)={
    val tmp=arr(a)
    arr(a)=arr(b)
    arr(b)=tmp
  }


  def main(args: Array[String]): Unit = {
    // 测试
    val arr=Array(5, 2, 9,11,3,6,8,4,0,0)
    val arrNew=quicksort(arr,0,arr.size-1)
    println(arrNew.toList.mkString(","))
    //0,0,2,3,4,5,6,8,9,11

  }
}



```
 


#### 联系邮箱 xxx_xxx@aliyun.com