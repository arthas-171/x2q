#  时间复杂度的概念
### [go back](/x2q/algorithm/algorithm)      
### [go home](/x2q)       
## 时间复杂度的概念
时间复杂度是指经过多少次计算(操作),可以实现目标效果,例如在集合元素获取中,指操作多少次集合可以拿到目标元素  
例如:对一个有n个元素的集合
O(1):表示操作一次即可获取目标元素
O(n):表示操作n次,即逐一遍历才可以获取目标元素
## O(log n) 
另一种常见的时间复杂度是 O(log n), 对数函数log
**对数的定义:**如果ax =N（a>0，且a≠1）,(注意这里是a的x次方)那么数x叫做以a为底N的对数，记作x=logaN，读作以a为底N的对数，其中a叫做对数的底数，N叫做真数。
## 以二分法为例
二分法的最坏时间责度是 O(log2n),最好是O(1),我们以最坏为例
加入我们有一个 16个有序元素的数组如下
![图片](/static/img/get1.png)  
我们要查找的元素是13,我们需要折半4次,及 16*(1/2),8*(1/2),4*(1/2),2*(1/2),才可以得到我们的目标元素
具体过程如下
![图片](/static/img/get2.png)  
![图片](/static/img/get3.png)  
![图片](/static/img/get4.png)  
![图片](/static/img/get5.png)  
![图片](/static/img/get6.png)  
总结如下
![图片](/static/img/get7.png)  
如果有n个元素就是
![图片](/static/img/get8.png)  
进一步推算入如下
![图片](/static/img/get9.png)
![图片](/static/img/get10.png)
![图片](/static/img/get11.png)
n为集合长度,k为操作次数,我们现在要求k,即对k求对数函数为  k=log2n
![图片](/static/img/get12.png)
![图片](/static/img/get13.png)
![图片](/static/img/get14.png)
以2为底数,log 1024的对数,计算结果是10

## 常见算法的时间复杂度
+ 冒泡排序 n*n
+ 二分法查找 log(n)
+ 快速排序法 最好n*log(n), 最坏 n*n

#### 联系邮箱 xxx_xxx@aliyun.com