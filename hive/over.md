# hive 分析函数及用法
### [go back](/x2q/hive/hive)      
### [go home](/x2q)     
 
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
+ ntile 切片函数,会按照某个列,讲所有数据均匀的切成n片
#### over语句 两种用法
+ over(order by col) 按照col排序累计,order by是一个默认的潜在窗口函数
+ over(partition by col ) 按照col 分区返回一个数据集
+ over(partition by col1 order by col2) 按照 col1分区并且按照col2 排序
#### over连用 row_number(), rank(),dense_rank()
+ row_number() 返回行号 如果有重复只会返回一个值
+ rank() 会返回全部排名 包括并列排名,它和dense_rank()区别是 rank是跳跃排 如果有两个并列第一那么下一个就直接是第三,dense_rank()是连续拍 两个并列第一下一个还是第二

**聚合函数 sum() max() avg() 和over(partition by col1 ) 连用可以统计分区的数量**

## 利用窗口函数 统计连续登陆
[参考连接](https://blog.csdn.net/TomAndersen/article/details/106432890)
表结构  
![图片](/static/img/get5.png)  
![图片](/static/img/get6.png)  
统计这个sql的思想就是,利用日期减去行号,因为日期是递增的+1,行号也是递增+1的,所以相减的差应该是一样的  
我们先写一个简单的统计每个人分组的行号的sql,这里有个问题要注意一下,如果有一个人一天多条重复登录记录的情况,无论采用那一种排名函数,
都无法做到隔天的连续登录,因此我们先去重复,直接group by name login_date/或者用行号取行号等于1去重复也行
![图片](/static/img/get1.png)  
输出结果
![图片](/static/img/get2.png)  
我们进行去重复之后的结果是
之后我们进行一个group by 分组的统计,根据name和diff分组,count(id)就可以知道连续登录的天数,因为如果是连续登录diff的值是一样的,顺便可以算出最早登录日期,最晚登录日期
![图片](/static/img/get3.png)  
结果
![图片](/static/img/get4.png)  
## 额外 实例如果应用 sum聚合 和 over的话 是会对条求和,如果我们想要最大的应该直接 group by 而不是 over (partition) 如下
![图片](/static/img/get7.png)  
![图片](/static/img/get8.png)  
[参考连接](https://blog.csdn.net/sherri_du/article/details/53312085)

## ntile() 切片函数
这个函数可以按照某一列将数据切成若干片,例如我们要统计店铺的销售额,
我们想统计销售额前30%的店铺的平均销售额,和后70%的平均销售额   
原始数据如下
![图片](/static/img/get9.png)  
核心思想是,按照价格递减切成10分, 123就是前30%,剩下的就是后70%,sql如下
```sql
-- 1 把记录按价格顺序拆分成10片
drop table if exists test_dp_price_rk;
create table test_dp_price_rk
as
select
 id,
 price,
 NTILE(10) OVER (order by price desc) as rn
from test_dp_price;

-- 2 按片取30%和70%，分别计算平均值
select
  new_rn,
  max(case when new_rn=1 then 'avg_price_first_30%' when new_rn=2 then 'avg_price_last_70%' end) as avg_price_name,
  avg(price) avg_price
from 
(
  select 
    id,
    price,
    rn,
    case when rn in (1,2,3) then 1 else 2 end as new_rn
  from test_dp_price_rk
)a
group by new_rn;
```
                                                          
                                                          
结果
![图片](/static/img/get10.png)        



rollup、cube、grouping sets  grouping_id                                                    
#### 联系邮箱 xxx_xxx@aliyun.com

