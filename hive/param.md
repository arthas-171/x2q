# hive 常用参数
### [go back](/x2q/hive/hive)      
### [go home](/x2q)      
 
##### 解决数据倾斜，负载均衡      如果没有数据倾斜不要加浪费资源，比如groupby带手机号
set hive.groupby.skewindata=true;
##### 设置reduce最大个数 (reduce端数据量大可以不加，比如最终结果在600g以上，通过 hive.exec.reducers.bytes.per.reducer 限制reduce个数)
set hive.exec.reducers.max=300;
##### job名
set mapreduce.job.name=P_DWA_D_IA_test;
##### 队列名
set mapreduce.job.queuename=ia;
##### cli窗口带scheme
set hive.cli.print.header=true; 
##### map预聚合  
set hive.map.aggr=true;
##### map内存
set mapreduce.map.memory.mb=2048;
##### reduce
set mapreduce.reduce.memory.mb=4096;
##### 根据map端shuffle输出数据量  判断多大数据量启用一个reduce
##### 而且reduce个数决定输出数据的文件数
set hive.exec.reducers.bytes.per.reducer=2147483648;
##### 是否对最终输出数据压缩
set hive.exec.compress.output=true;
##### 最终输出数据压缩方式    压缩方式根据需求改变    snappy  gz lz4        org.apache.hadoop.io.compress.SnappyCodec
set mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec;
##### 最终输出数据压缩类型
set mapred.output.compression.type=BLOCK;
##### 中间shuffle数据是否压缩
set hive.exec.compress.intermediate=true;
##### 中间shuffle数据压缩方式
set hive.intermediate.compression.codec=org.apache.hadoop.io.compress.SnappyCodec;
##### 中间shuffle数据压缩类型
set hive.intermediate.compression.type=BLOCK;
##### 在Map-only的任务结束时合并小文件
set hive.merge.mapfiles=true;
##### 在Map-Reduce的任务结束时合并小文件
set hive.merge.mapredfiles=true;
##### 当输出文件的平均大小小于该值时，启动一个独立的map-reduce任务进行文件merge
set hive.merge.smallfiles.avgsize=134217728;
##### 合并文件的大小
set hive.merge.size.per.task=536870912;
##### 决定每个map处理的最大文件大小(结合CombineHiveInputFormat可以合并小文件)
set mapred.max.split.size=1073741824;
set mapred.min.split.size.per.node=1073741824;
set mapred.min.split.size.per.rack=1073741824;
##### 决定输入格式    合并小文件(或者block)
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;
##### 是否支持可切分的CombineInputFormat 合并输入小文件此参数必须加否则不生效
set hive.hadoop.supports.splittable.combineinputformat=true;







#### 联系邮箱 xxx_xxx@aliyun.com

