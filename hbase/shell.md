# hbase shell 命令
### [go back](/x2q/hbase/hbase)      
### [go home](/x2q)       
## 命令
+ list：显示表
+ get 'tableName','rowkey' 查询单条
+ scan 'tableName' ,{LIMIT=>10}  扫描10条
+ desc 'tableName'  查看表结构        
+ scan 'hbase:meta',{STARTROW=>'tableName,rowkey,9999999999999',REVERSED=>true,LIMIT=>1}，这是查询某一条rowkey所在的region
扫描的是hbase的元数据表，开始的rowkey就是目标rowkey，截止rowkey可以是无限大，REVERSED是是否翻转，因为不翻转查出来的会是下一个，
实例如图
![图片](/static/img/get4.png)                         
查询rowkey为27503101551这个rowkey所在的region，结果显示它在startkey为25，截止key为30的region中，符合我们的预期
+ get 't1', 'r1', {COLUMN => 'c1'} 查询单列
+ get 't1', 'r1', 'f1' 查询单个列族 
+ create 'ubdXlRealtime_ssgj','f' ,SPLITS=>['05','10','15','20','25','30','35','40','45','50','55','60','65','70','75','80','85','90','95'] 建表并且预设region
#### 联系邮箱 xxx_xxx@aliyun.com