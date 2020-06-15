# hdfs 原理

##概述:
Hadoop Distributed File System(HDFS):是一个高吞吐量的分布式文件系统.是分布式计算的基础

##基本概念:
NameNode (元数据节点):存储元数据信息,包括 fsimage(命名空间镜像映像),edits_log(操作日志),NameNode支持主备HA,
NameNode会在内存维护命名空间,并且在磁盘上生成 命名空间映射文件fsimage和日志操作文件 edits_log,命名空间存储了数据块(block)和DataNode的对应关系
              
**我们可以查看 NameNode在磁盘上生成的文件**
![日志示例](/static/img/165511_JHWG_2969788.png)