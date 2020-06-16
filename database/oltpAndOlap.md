# OLTP和OLAP 在线事务处理和在线分析处理
### [go back](/database.md)      
### [go home](../README.md)     
## oltp和olap
oltp和olap是两种常见的业务模式,  
**oltp**(on line transcation processing) 在线事务处理 它具有以下特点,常见的系统如银行财务
+ 实时性要求高
+ 数据量相对较小
+ 要求绝对的事务完整性,增删改查操作一般都会涉及
+ 并发量高
  
**olap**(on line analytical processing) 在线分析处理,这种业务一般是日志的分析和深度挖掘,常见如淘宝的交易记录,百度地图的人口迁徙记录 具有以下特点,olap的结果一般是为决策提供支持
+数据量通常非常大
+对事务要求不高 通常只有 添加和查询操作
+实时性要求不高通常只有汇总后的结果,而汇总分析过程通常可以执行很久


#### 联系邮箱 xxx_xxx@aliyun.com

