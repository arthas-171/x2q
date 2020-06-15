# hdfs 原理
### [go back](/hdfs.md)      
### [go home](../README.md)     
##概述:
Hadoop Distributed File System(HDFS):是一个高吞吐量的分布式文件系统.是分布式计算的基础

##基本概念:
NameNode (元数据节点):存储元数据信息,包括 fsimage(命名空间镜像映像),edits_log(操作日志),NameNode支持主备HA,
NameNode会在内存维护命名空间,并且在磁盘上生成 命名空间映射文件fsimage和日志操作文件 edits_log,命名空间存储了数据块(block)和DataNode的对应关系
              
**我们可以查看 NameNode在磁盘上生成的文件**
![日志示例](/static/img/165511_JHWG_2969788.png)  
(这些文件的路径是  hdfs 配置文件的 NameNode 的工作目录 )  
**Secondary NameNode**(辅助元数据节点):并不是NameNode的备份,而是 NameNode的辅助节点,负责合并 "命名空间镜像映射文件"和"日志操作"文件,然后生成新的 "命名空间镜像"发送给NameNode 
辅助元数据节点周期性的合并文件,可以防止"日志文件"过大,合并后的命名空间镜像 会在辅助namenode 节点保存一份,在元数据库节点失败的时候可以恢复  
**DataNode(数据节点)**:负责存储数据块(block),周期性的向元数据节点汇报数据块信息  
**block(数据块)**:文件会被切分成多个数据块block进行存储,block默认是64MB,每个block会在多个DataNode节点上存储备份,默认是3份,可以通过配置设置备份个数  
##整体架构图
![架构图](/static/img/175900_voia_2969788.png)  
##Secondary NameNode 的工作原理
当 一个 client 进行 写操作的时候 首先会它会记录在 edit log中,之后 namenode会修改 内存总的"命名空间", fsimage 文件 是 "命名空间映像文件",它是内存中的元数据 ,"命名空间"会被定时的checkpoint(检查点),最后会在硬盘上生成 fsimage文件,fsimage文件是一种序列化格式不能直接修改 
如果元数据节点失败那么会把checkpoint(检查点)的元数据信息fsimage 加载到内存中辅助元数据节点（secondary NameNode)是帮助 元数据节点(NameNode)将 内存中的 元数据信息 checkpoint到 硬盘上 ,操作比较费时 所有让从 元数据节点来做
       
##checkpoint 具体操作
+ 辅助元数据节点（secondary NameNode)通知 元数据节点(NameNode)生成 新的 日志文件,并且以后的日志都写入到新的日志文件中
+ 辅助元数据节点(secondary NameNode)http协议从元数据节点(NameNode)获取 fsimage文件和旧的日志文件
+ 辅助元数据节点(secondary NameNode)合并 fsimage和日志文件 生成新的fsimage文件
+ 辅助元数据节点(secondary NameNode)将新的fsimage文件传回 元数据节点(NameNode)
+ 元数据节点(NameNode) 将旧的fsimage替换成新的,日志文件也替换成新的(在第一步时生成的)
##读取文件过程:
+ 客户端(client)用FileSystem的open()函数打开文件
+ DistributedFileSystem用RPC调用元数据节点，得到文件的数据块信息。
+ 对于每一个数据块，元数据节点返回保存数据块的数据节点的地址。
+ DistributedFileSystem返回FSDataInputStream给客户端，用来读取数据。
+ 客户端调用stream的read()函数开始读取数据。
+ DFSInputStream连接保存此文件第一个数据块的最近的数据节点。
+ Data从数据节点读到客户端(client)
+ 当此数据块读取完毕时，DFSInputStream关闭和此数据节点的连接，然后连接此文件下一个数据块的最近的数据节点。
+ 当客户端读取完毕数据的时候，调用FSDataInputStream的close函数。
+ 在读取数据的过程中，如果客户端在与数据节点通信出现错误，则尝试连接包含此数据块的下一个数据节点。
+ 失败的数据节点将被记录，以后不再连接。
##写入文件
+ 客户端调用create()来创建文件
+ DistributedFileSystem用RPC调用元数据节点，在文件系统的命名空间中创建一个新的文件。
+ 元数据节点首先确定文件原来不存在，并且客户端有创建文件的权限，然后创建新文件。
+ DistributedFileSystem返回DFSOutputStream，客户端用于写数据。
+ 客户端开始写入数据，DFSOutputStream将数据分成块，写入data queue(数据队列)。
+ Data queue由Data Streamer读取，并通知元数据节点分配数据节点，用来存储数据块(每块默认复制3块)。分配的数据节点放在一个pipeline里。
+ Data Streamer将数据块写入pipeline中的第一个数据节点。第一个数据节点将数据块发送给第二个数据节点。第二个数据节点将数据发送给第三个数据节点。
+ DFSOutputStream为发出去的数据块保存了ack queue，等待pipeline中的数据节点告知数据已经写入成功。
###如果数据节点在写入的过程中失败

+ 关闭pipeline，将ack queue中的数据块放入data queue的开始。
+ 当前的数据块在已经写入的数据节点中被元数据节点赋予新的标示，则错误节点重启后能够察觉其数据块是过时的，会被删除。
+ 失败的数据节点从pipeline中移除，另外的数据块则写入pipeline中的另外两个数据节点。  
+ 元数据节点则被通知此数据块是复制块数不足，将来会再创建第三份备份。  
+ 当客户端结束写入数据，则调用stream的close函数。此操作将所有的数据块写入pipeline中的数据节点，并等待ack queue返回成功。最后通知元数据节点写入完毕。