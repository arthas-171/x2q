#  bloom 布隆过滤器
### [go back](/x2q/algorithm/algorithm)      
### [go home](/x2q)   
## 概述
   bloom filter是1970年由Burton Howard Bloom提出的,本质上是一个很长的二进制向量和一些列hash映射函数
+ 优点:空间效率和时间查找效率非常非常高
+ 缺点:存在一定误判概率,即**假阳性**,它会返回两种结果,一种是**可能存在**,一种是**一定不存在**
## 实现原理
  如果我们想要判断一个元素是否在一堆元素之中,或者是做过滤判断这个元素是否出现过,那么我们肯定需要把已经出现过的元素保存起来,
保存方式有**链表**,**数组**,**树**,他们的查找的时间复杂度分别是 链表是O(n),数组是O(1),树是O(log n),显而易见数组(hash表)
查找效率是最高的,但是无论如何这三个的存储效率都不高,所以我们需要解决的问题关键变成了如何存储元素.bloom用二进制数组来高效存贮
数据,注意存储的不是数据本身而是数据的一个变向的映射,带来的额外好处是保密性,坏处是这个映射可能存在重复,即返回存在是**可能存在**
     
   假设数组长度是m,经由k个hash函数计算,添加一个元素时,k个hash函数计算出的索引值 h1(x), h2(x)，… hk(x)被设置位1,检查是否存在
时只需要检查这几个位置点的值是否全是1即可,但是因为不同元素是有一定概率会被映射到相同位置点的,所以存在误报.
## 计算公式
### 布隆过滤器的误报概率
假设数组长度为m,并使用k个hash函数，n是要插入到过滤器中的元素个数，那么误报率的计算如下：
![图片](/static/img/get15.png)  
### 位数组的大小
如果过滤器中的元素数量已经知道，期望的误报率位p，那么二进制位数组大小计算公式如下：
![图片](/static/img/get16.png)  
### 最优哈希函数数量
如果m是数组长度，n是插入的元素个数，k是hash函数的个数，k计算公式如下如果m是数组长度，n是插入的元素个数，k是hash函数的个数，k计算公式如下
![图片](/static/img/get17.png)  
### 在线计算地址
当然了 我们不想自己算,这里有一个在线计算地址,感谢社区
https://krisives.github.io/bloom-calculator/
## 如何应用
直接使用 google提供的版本,
````java
<!-- https://mvnrepository.com/artifact/com.google.guava/guava -->
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>30.1-jre</version>
</dependency>
````
```scala
package cn.x2q.algorithm

import java.util.UUID

import com.google.common.hash.{BloomFilter, Funnels}

/**
  * @Auther: huowang
  * @Date: 11:53:29 2021/1/30
  * @DES: 未完待续
  * @Modified By:
  */
object TestBloom {

  def main(args: Array[String]): Unit = {
     val filter:BloomFilter[String] = BloomFilter.create(
      Funnels.integerFunnel(),
       1000,
      0.0000001);

    var flag=1
    var count=0
    while (flag<1000){
      val uuid=UUID.randomUUID().toString()
      filter.put(uuid)
      if(filter.mightContain(uuid)){
        count=count+1
      }
      flag=flag+1
    }
    println("============>"+count)
  }
}

```

#### 联系邮箱 xxx_xxx@aliyun.com