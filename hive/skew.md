# hive 数据倾斜
### [go back](/x2q/hive/hive)      
### [go home](/x2q)     
 
## hive数据倾斜
引起数据倾斜的操作  
+ join 一个表较小,但是key集中,分配到某一个reduce或者几个reduce上的数据远远高于平均值
+ join 大表与大表join,但是分桶操作的字段为0或者null,结果就是这些空值都由同一个reduce处理,最终导致任务非常慢
+ group by 维度过小 导致某些值数量过多,处理某值的reduce非常耗时
+ count distinct  某特殊值过多 导致处理此值的reduce非常耗时

## 解决方式
### 参数调节
 hive.map.aggr=true  
 map端部分聚合 相当于combiner  
 hive.groupby.skewindata=true   
  
有数据倾斜的时候进行负载均衡，当选项设定为 true，生成的查询计划会有两个 MR Job。第一个 MR Job 中，Map 的输出结果集合会随机分布到 Reduce 中，每个 Reduce 做部分聚合操作，并输出结果，这样处理的结果是相同的 Group By Key 有可能被分发到不同的 Reduce 中，从而达到负载均衡的目的；第二个 MR Job 再根据预处理的数据结果按照 Group By Key 分布到 Reduce 中（这个过程可以保证相同的 Group By Key 被分布到同一个 Reduce 中），最后完成最终的聚合操作

### sql 语句调节
#### 如何Join：
关于驱动表的选取，选用join key分布最均匀的表作为驱动表  
做好列裁剪和filter操作，以达到两表做join的时候，数据量相对变小的效果。
#### 大小表Join：
使用map join让小的维度表（1000条以下的记录条数） 先进内存。在map端完成reduce.
#### 大表Join大表：
把空值的key变成一个字符串加上随机数，把倾斜的数据分到不同的reduce上，由于null值关联不上，处理后并不影响最终结果。
#### count distinct大量相同特殊值
count distinct时，将值为空的情况单独处理，如果是计算count distinct，可以不用处理，直接过滤，在最后结果中加1。如果还有其他计算，需要进行group by，可以先将值为空的记录单独处理，再和其他计算结果进行union。
#### group by维度过小：
采用sum() group by的方式来替换count(distinct)完成计算。
#### 特殊情况特殊处理：
在业务逻辑优化效果的不大情况下，有些时候是可以将倾斜的数据单独拿出来处理。最后union回去

## 补充 group by 和distinct 区别
group by 和distinct 都可以达到去重的目的, 但是 group by 本质上是分组, 比如又一张网站访问记录表,有用户账号 访问时间 等字段,现在需要统计某一天的全部访客账号,
+ select userId from visit_log where day='2018-01-30' group by userId;
+ select distinct userId from  visit_log where day='2018-01-30' ;
这两个查询出的结果是一样的但是执行逻辑完全不一样,distinct的作用是返回某一字段的不重复记录 ,如果我们想统计莫一天的访客数量那么可以写作
+ select count(distinct userId) from  visit_log where day='2018-01-30';  
distinct 只会生成一个reduce进行结果统计,所以速度回非常慢,但是如果数据量小的话不会又太大影响  
![图片](/static/img/01ad127c713862336cd40f1b084149879b1.jpg)    
select count(t.userId)  from(select userId from visit_log where day='2018-01-30' group by userId) t;  
![图片](/static/img/178a1506e0587ced48742a852b9b16d6b22.jpg)    
group by 会生成多个reduce任务 同时执行会比一个reduce要快,而且数据量越大效果越明显  

[参考](https://blog.csdn.net/s646575997/article/details/51510661)

#### 联系邮箱 xxx_xxx@aliyun.com

