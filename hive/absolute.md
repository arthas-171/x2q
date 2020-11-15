# hive sql 和 spark sql 执行结果不一致问题
### [go back](/x2q/hive/hive)      
### [go home](/x2q)   
 
## hive sql 进行比较例如 join条件 t1.a=t2.b 执行结果不符合预期
可能的原因是 a列的类型和b列的类型不一致引起的,例如a是double,b是string,  
解决方案 比较之前强转类型 cast(t1.a AS String)=cast(t2.b AS String),   
但是 spark sql在这方便兼容性好,一般可以执行成功
## 对于 parquet/orc格式文件 hive sql 和 spark sql执行结果不一致
在 spark 2.3.2的版本中 spark sql尝试使用自己的解码方式读取 parquet格式文件,以便获得
更高的效率,但是由此可能带来结果的不一致性    
**解决方案** 
+ config("spark.sql.hive.convertMetastoreParquet","false")
+ config("spark.sql.hive.convertMetastoreOrc","false")   
**参数解释** spark.sql.hive.convertMetastoreParquet默认设置是true, 
它代表使用spark-sql内置的parquet的reader和writer(即进行反序列化和序列化),
它具有更好地性能，如果设置为false，则代表使用 Hive的序列化方式,Orc同理


#### 联系邮箱 xxx_xxx@aliyun.com

