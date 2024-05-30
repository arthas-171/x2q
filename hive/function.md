# hive 常用函数
### [go back](/x2q/hive/hive)      
### [go home](/x2q)      
 
## 常用函数
+ COALESCE(T v1, T v2,…):返回参数中第一个非null值 ,如果全是null 返回null
+ collect_set/collect_list:列转行 去重/不去重, 配合concat_ws(',',collect_set(userid)) 将集合转为逗号分隔的字符串
+ explode:行转列,对于数组类型,map类型可以行转列,如下,但是只能查询 c1,想要把其他列查出来需要配合 lateral view 
                                   
|  c1   | 
|  ----  | 
| [1,2,3]  | 
| [4,5,6] | 
                                
select explode(c1) from t  ;
                                    
| c1 | 
| ---| 
| 1  | 
| 2  | 
| 3  | 
| 4  | 
| 5  | 
| 6  | 
                                      
+ lateral view explode: 本质上是将集合拆分成一个表,然后左连接剩下的列的表
                               
|  id   | c1  |
|  ----  | ----  |
| a  | [1,2,3] |
| b  | [4,5,6] |
                                    
select id,cc1 from t lateral view explode (c1) ct as cc1;
                                         
|  id   | cc1 |
|  ----  |-----|
| a  | 1   |
| a  | 2   |
| a  | 3   |
| b  | 4   |
| b  | 5   |
| b  | 6   |
  
+ nvl 函数,把空值转换成一个其他值,例如 nvl(user_name,'null')                                     


## 抽样的三种方式

### 数据块抽样（tablesample()函数）
1） tablesample(n percent) 根据hive表数据的大小按比例抽取数据，并保存到新的hive表中。如：抽取原hive表中10%的数据
（注意：测试过程中发现，select语句不能带where条件且不支持子查询，可通过新建中间表或使用随机抽样解决）
create table xxx_new as select * from xxx tablesample(10 percent)
2）tablesample(n M) 指定抽样数据的大小，单位为M。
3）tablesample(n rows) 指定抽样数据的行数，其中n代表每个map任务均取n行数据，map数量可通过hive表的简单查询语句确认（关键词：number of mappers: x)

### 分桶抽样
hive中分桶其实就是根据某一个字段Hash取模，放入指定数据的桶中，比如将表table_1按照ID分成100个桶，其算法是hash(id) % 100，这样，hash(id) % 100 = 0的数据被放到第一个桶中，hash(id) % 100 = 1的记录被放到第二个桶中。创建分桶表的关键语句为：CLUSTER BY语句。
分桶抽样语法：
TABLESAMPLE (BUCKET x OUT OF y [ON colname])
其中x是要抽样的桶编号，桶编号从1开始，colname表示抽样的列，y表示桶的数量。
例如：将表随机分成10组，抽取其中的第一个桶的数据
select * from table_01 tablesample(bucket 1 out of 10 on rand())

### 随机抽样（rand()函数）
1）使用rand()函数进行随机抽样，limit关键字限制抽样返回的数据，其中rand函数前的distribute和sort关键字可以保证数据在mapper和reducer阶段是随机分布的，案例如下：
select * from table_name where col=xxx distribute by rand() sort by rand() limit num;
2）使用order 关键词
案例如下：
select * from table_name where col=xxx order by rand() limit num;
经测试对比，千万级数据中进行随机抽样 order by方式耗时更长，大约多30秒左右。

### 参考链接：https://blog.csdn.net/zylove2010/article/details/78290319


#### 联系邮箱 xxx_xxx@aliyun.com

