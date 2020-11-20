# hbase 基础概念
### [go back](/x2q/hbase/hbase)      
### [go home](/x2q)       
## 基本概念

###  概要
rowkey是一行的唯一表示,rowkey的设计重点要考虑业务场景,需要避免两种情况,一是写入热点,二是查询热点,针对查询又要兼顾查询效率

### 查热点
**查热点:** 顾名思义就是查询的操作集中落在了某一个或者少数的region上面,进而造成只由集群中一个或者少数几个regionServer处理读取操作,
导致这些机器的io负载过高,   
首先讨论这个问题之前需要明确hbase的查询方式,hbase支持根据rowkey获取单条数据的get方式,和扫描表scan方式,

#### get 方式
 **单条get:** 如果我们的业务场景是码表匹配,比如又一个全国的用户信息表,每次进行匹配的时候,需要查询该表,但是每次匹配的数据大多来自某一个省,少数来自
 其他省市,但是是逐条查询高并发访问,这样我们需要设计rowkey的时候要让全国的数据尽量散列到各个分区,同一个省,市,区的人 要尽量不在同一个region,
 每个省的人 均为的分布在每个region上面,
 **get list:** 对于批量更加rowkey获取数据,hbase有一种getList的方式, 先将rowkey转换成get对象,放入get对象的集合,再将getlist集合整体提交查询,   
 实例代码
                                                          
                                                          
```java
public static List<String> queryTableTestBatch(List<String> rowkeyList) throws IOException {
        List<Get> getList = new ArrayList();
        List<String> list = new ArrayList();
        String tableName = "table_a";
        Table table = connection.getTable( TableName.valueOf(tableName));// 获取表
        for (String rowkey : rowkeyList){//把rowkey加到get里，再把get装到list中
            Get get = new Get(Bytes.toBytes(rowkey));
            getList.add(get);
        }
        Result[] results = table.get(getList);//重点在这，直接查getList<Get>
        for (Result result : results){//对返回的结果集进行操作
            for (Cell kv : result.rawCells()) {
                String value = Bytes.toString(CellUtil.cloneValue(kv));
                list.add(value);
            }
        }
        return list;
    }
```                                                          
             
             
对于这种场景批量查询的场景我们应该尽量把rowkey放在一起  

#### scan方式
这种方式是批量的扫描表, scan支持以下条件
+ COLUMNS
+ Limit 限制查询结果行数
+ STARTROW ROWKEY 起始行。会先根据这个 key 定位到 region，再向后扫描
+ STOPROW 结束行
+ TIMERANGE 限定时间戳范围
+ VERSIONS   版本数   
+ FILTER   按条件过滤行                        
例子 表名 test_Table
+ scan 'test_Table',{LIMIT=>10,FILTER=>"PrefixFilter('zhangsan')"}
+ scan 'test_Table',{STARTROW=>'b',STOPROW='e',LIMIT=>10}
+ scan 'zy_comment',{LIMIT=>10,FILTER=>"ValueFilter(=,'substring:666')"}
+ scan 'zy_comment',{LIMIT=>10,FILTER="SingleColumnValueFilter('info','articleType',=,'binary: \x00\x00\x00\x00\x00\x00\x00\x02')"}
+ scan 'zy_comment',{FILTER=>"RowFilter(=,'substring:zhangsan')"}

scan的方式最好让相似的数据集中在一个region里面,比如还是全国的用户信息,我们每次批量取一个区的用户信息装在入内存,这种情况,我们可以用 省_市_区_userId,作为rowkey
每次scan,扫描的数据大致会在一个region或者几个region,集中获取这些数据效率比较高,
#### 联系邮箱 xxx_xxx@aliyun.com