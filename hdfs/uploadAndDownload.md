# hdfs上传/下载文件过程详解 
### [go back](/hdfs.md)      
### [go home](../README.md)     
## hdfs上传文件过程详解
+ client端通知namenode要上传文件,namenode检查文件名是否已经存在,如果不存在通知可以上传,并且返回可以用于存储的datanode列表
+ client 切割文件为block块(默认大小128MB),向namenode请求上传block1,namenode返回可用的DataNode信息,
+ client和返回的第一个DataNode1建立通信管道(pipeline),之后开始上传文件内容,然后DataNode1会和DataNode2同步数据,DataNode2和DataNode3同步数据,
+ 当同步数据结束之后通知client上传完毕,client通知nameNode 块1上传完毕,NameNode会记录元数据信息(block1存在的DataNode位置)
+ 时候进行block2的上传
+ 注意 ,如果 client和DataNode的pipeline断开则会认为上传失败,需要重新上传,如果DataNode1和DataNode2之间的pipeline断开但是已经有一个完整的文件上传结束的话,认为上传成功,namenode后续会指定其他DataNode域DataNode1同步副本数
## hdfs 下载文件过程详解
+ client 向Namenode提交get下载文件请求
+ NameNode返回相关数据的block list块列表,这个列表是排序过的,排序原则是根据距离client所在机器的路由远近,和心跳检测
+ client获取理他最近的一个block的地址,与相关的DataNode建立通讯
+ 通讯是socket stream 重复调用父类的dataInputStream的read方法指导读取完block的全部数据
+ 读取完毕一个block会进行行数校验check num,如果发现和NameNode记录的行数不一致会重新请求NameNode给一个新的block地址进行读取
+ 会并行读取多个block块,最后合并成一个文件

#### 联系邮箱 xxx_xxx@aliyun.com