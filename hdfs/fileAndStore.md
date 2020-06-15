#hdfs 文件存储格式
### [go back](/hdfs.md)      
### [go home](../README.md)     
**hdfs 文件存储格式分为两大类 行存储和列存储**    
  
**行存储**,将一整行存储在一起,是一种连续的存储方式,例如SequenceFile,MapFile,缺点是如果只需要行中的某一列也必须把整行都读入内存当中    
  
**列存储** 列存储会把文件切割成若干列,每一列存储在一起,是需要那一列读取那一列,不需要的不用读取,例如parquet ORCfile,RCfile,列存储不适合流式写入,写入失败当前文件无法恢复因此flume采用行存储,列存储由于每一列中的数据类型相同所以可以根据数据类型选择适合的编码和压缩格式    
  
**SequenceFile**:Hadoop提供的一个*行存*储结构,Hadoop适合处理大文件而不适合处理小文件,所以sequencefile是为小文件提供的一种容器,将小文件包装起来形成一个SequenceFile类, 它用一种<key,value>的形式序列化数据导文件中    
  
**MapFile**:MapFile可以看做有序的SequenceFile,是排过序的SequenceFile,它有索引可以按照索引查找,索引作为一个单独的文件存储,一般128个记录存储一个索引,索引可以载入内存,方便快速查找    
  
hdfs 最开始只有行存储的这两种形式 SequenceFile和macFile,除此之外还有text文本,但是之后再hive中丰富了存储结构包括如下几种    
  
**RCFile**:hive的RCfile 是将数据按照行分组 ,组内在按照列划分储存    
 
**ORCfile**:是RCfile的升级版,将数据划分为默认大小为250MB的stripe(条带),每个stripe包含索引,数据和footer,ORCfile包换索引比RCfile更加高效    
  
**Parquet**:parquet基于Google的dremel,擅长处理深度嵌套的数据(有点类似于嵌套多层的json格式),parquet会将嵌套结构整合为平面列存储,    


