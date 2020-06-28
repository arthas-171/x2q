# hive 分析函数及用法
### [go back](/hive.md)      
### [go home](../README.md)    
 
## hive 分析函数及用法
### hive 常用于olap(On-Line Analytical Processin 在线分析处理)领域,如下,分析函数不同于聚合函数,聚合函数会返回一个值但是分析函数会返回一个数据集,通过配合可以对数据集进行分析  

+ 分区排序
+ 动态group by
+ 求 Top n
+ 累计计算
+ 层次查询
#### 窗口函数和over (partition by col1 oder by col2) 连用
+ first_value 分组内排序后 截止到当前的第一个值
+ last_value 分组内排序后 截止到当前的最后一个值
+ lead(col,n,default) 用于获取分组内向下的第n行,第一个参数是列名,第二个是向下第几行,第三个是可选默认值,如果为null的时候就用默认值顶替
+ lag(col,n,default) 和lead相反分组内向上去第n行的值   
#### over语句 两种用法
+ over(order by col) 按照col排序累计,order by是一个默认的潜在窗口函数
+ over(partition by col ) 按照col 分区返回一个数据集
+ over(partition by col1 order by col2) 按照 col1分区并且按照col2 排序
#### over连用 row_number(), rank(),dense_rank()
+ row_number() 返回行号 如果有重复只会返回一个值
+ rank() 会返回全部排名 包括并列排名,它和dense_rank()区别是 rank是跳跃排 如果有两个并列第一那么下一个就直接是第三,dense_rank()是连续拍 两个并列第一下一个还是第二

**聚合函数 sum() max() avg() 和over(partition by col1 ) 连用可以统计分区的数量**

[参考连接](https://blog.csdn.net/sherri_du/article/details/53312085)


#### 联系邮箱 xxx_xxx@aliyun.com

