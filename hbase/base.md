# hbase 基础概念
### [go back](/x2q/hbase/hbase)      
### [go home](/x2q)       
## 基本概念
+ 表是行的集合
+ 行是列族的集合
+ 列族是列的集合
+ 列是键值的集合
![图片](/static/img/get1.png)   

![图片](/static/img/get2.jpg)   
![图片](/static/img/get3.jpg)       
                                                              
## 详述
###  rowkey
rowkey和关系型数据库的主键含义一样,用于标识唯一的一行,最大长度64KB，实际应用中长度一般为10-100bytes,rowkey存储时基于字典排序,
+ 先 rowkey 升序排序，
+ rowkey 相同则 column key 升序排序
+ rowkey、column key 相同则 timestamp 降序排序
所以经常需要**一起**读取的rowkey应当设计的时候前缀相同让它们存储在一起,但是如果每次读取一条,应该尽量散列分散到不同region上,避免读热点
行的一次读写是原子操作（不论一次读写多少列）
 hbase 查询有基于rowkey的单条查询,基于rowkey的范围扫描,scan的扫描  

### column family 列族
列族是列的集合,一个rowkey下面可以有多个列族,但是不是越多越好,hbase推荐1-3个列族,hbase建表的时候必须制定列族,列族下面可以有多个列方便存取数据
列是支持动态创建的,不需要在建表的时候就指定,但是列族是需要在建表时就指定的,同一个列族下面的数据在物理存储上会存的相对靠近
### region 区
region和hive的分区差不多,一个表默认至于一个region,但是可以通过自定义的初始化划分多个region,hbase基于rowkey将数据分别储存到多个region里面
每个region负责一定范围的数据访问和存储,
### timestamp 版本(时间戳)
hbase通过timestamp来维护数据的版本,写入数据的时候可以指定版本,如果不指定hbase会自动添加当前服务器时间为当前版本,rowkey的数据按照timestamp
倒序排列,默认查询的就是最新的数据
                          
## hbase 架构
![图片](/static/img/get3.png)  
hbase 架构分为master和regionServer(多个)(分区服务),
### hbase master
master 是主控服务,负责以下功能
+ 对schema的操作(建表,删表,修改表)
+ 为regionServer分配region
+ 负责regionServer间的负责均衡
+ 发现失效的regionServer并且重新分配
+ HDFS上的垃圾回收
   
### regionServer
regionServer负责以下功能
+ 维护master给他分配的region,处理对这些region的读写请求
+ 负责切割随着程序运行不断变大的region                
**总结:** 可以发现对hbase的读写并不需要访问master,因此master的负载很低,读写通过zookeeper和regionServer进行寻址,之后直接与regionServer进行交互
master仅仅负责维护table和region的元数据信息,table的元数据信息保存在zookeeper上,
+ 一个表对应多个region,分布储存在多个regionServer上,
+ 表中每一个列族对应region上一个store,一个表有多少列族就有多少store
+ 每个store会有一个menstore和0或者多个storeFile,每个storeFile对应的是一个HFile,是数据真正存储的文件,    
+ 一个regionServer会有一个对应的log和多个region       
                                                                 
                                                                 
一个表在行方向上被分成多个region,region是负责负载均衡的最小单元,不同的region存在不同的regionServer上,如果表不预设region那么初始化的时候
只有一个region,会随着数据量的不断增加,而自行分裂,阈值是256MB,但是这样,如果rowkey是递增的会存在写热点的问题,只会往最新的region写数据,
### regionServer如何定位region
+ 通过zk里的文件/hbase/rs得到-ROOT-表的位置。-ROOT-表只有一个region
+ 通过-ROOT-表查找.META.表的第一个表中相应的HRegion位置。其实-ROOT-表是.META.表的第一个region；.META.表中的每一个Region在-ROOT-表中都是一行记录。
+ 通过.META.表找到所要的用户表HRegion的位置。用户表的每个HRegion在.META.表中都是一行记录
**总结:**-ROOT-表永远不会被分隔为多个HRegion，保证了最多需要三次跳转，就能定位到任意的region。Client会将查询的位置信息保存缓存起来，缓存不会主动失效，
因此如果Client上的缓存全部失效，则需要进行6次网络来回，才能定位到正确的HRegion，其中三次用来发现缓存失效，另外三次用来获取位置信息。           
+ -ROOT-：记录.META.表的Region信息。
+ .META.：记录用户表的Region信息
-ROOT-、.META.与其他表没有任何区别
### 关于 store
hbase把同一个列族里面的数据存在一个store里,因此我们应该把相关相似的数据放到一个列族里面,hbase是以store的大小来判断是否需要分裂region的
+ menStore 是在内存中的结构,保存修改的数据即keyValue,当到达64MB(默认),就会flush到文件里面,生成一个快照,hbase有一个独立的线程负责这个操作
+ StoreFile 就是menStore flush到磁盘的文件,StoreFile底层的存储格式HFile   
###  Hlog 
WAL意为write ahead log，用来做灾难恢复使用，HLog记录数据的所有变更，一旦region server 宕机，就可以从log中进行恢复      
+ LogFlusher:定期的将缓存中信息写入到日志文件中
+ LogRoller: 对日志文件进行管理维护                                                          
#### 联系邮箱 xxx_xxx@aliyun.com