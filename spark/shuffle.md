# spark shuffleManager
### [go back](/spark.md)      
### [go home](../README.md)     
## spark shufflemananger

+ hashShuffleManager spark1.2之后已经废弃了,除了面试官会问问没啥用,以前可以配置选择shuffleManager的具体实现,现在已经全部默认SortShuffleManager
+ SortShuffleManager:现在spark在用的shuffleManager实现


## HashShuffleManager

+ **最原始的HashShuffleManager**, 每上游stage会为下游stage每个都创建一个文件,根据key写入不同的文件,写入数据的时候先现在内存中缓存,达到阈值后溢出到文件.例如上游stage有100个task,下游stage有50个task,这样上游stage就会生成一共100*50个文件,带来大量的磁盘io,当然这5000个文件是均匀分散在每个executor上的
+ **经过优化的HashShuffleManager**,场景假设和上面一样,每个executor中的第一批task会创建shuffleFileGroup,也就是说他们会为下游每个task创建一个文件,后续的task复用第一批task创建的文件 ,在里面追加内容,这里面文件内容是分段的,数据是分段存储的,生成的文件数是  executor数*下游task数,

## SortShuffleManager shuffle write 阶段
### SortshuffleManager分为普通模式和bypass
#### 普通模式
shuffle write阶段首先将数据写入内存数据结构,(bykey的聚合算子写入map结构,join的非聚合算子写入array)达到阈值后写入磁盘临时文件,
写入时根据key排序,分批次写入每次1万条.这样减少磁盘io.当全部上游task都写完的时候会merge合并这些临时文件为一个文件并且建立一个索引文件,
索引文件中记录start offset 和end offset,无论下游有多少个task,下游task根据这索引文件read自己需要的数据.所以无论下游多少个task,
文件数永远是上游task个数个,比如上游有100个task,就有100个文件
#### bypas模式,
bypass模式需要满足两个条件(同时满足 spark源码注释原话是 If there are fewer than spark.shuffle.sort.bypassMergeThreshold partitions and we don't need map-side aggregation, )
+ shuffle write task数小于200(默认值),通过 spark.shuffle.sort.bypassMergeThreshold 设置
+ 不是聚合算子,非聚合算子例如 join,聚合算子例如 combinebykey,只要是bykey的都是聚合算子  
bypass模式和普通模式的区别  
+ 写磁盘机制不同,直接将数据按key的hash值写入临时磁盘文件,之后再将文件merge
+ 没有内存中排序

**sortShuffleMaanger 普通模式的图解**  

![图片](/static/img/up-a77db1169be9f1d1979e2dd4ac737c8e3fa.png)

## Shuffle Read
#### 在什么时候获取数据，Parent Stage 中的一个 ShuffleMapTask 执行完还是等全部 ShuffleMapTasks 执行完？
 当 Parent Stage 的所有 ShuffleMapTasks 结束后再 fetch(拉取)。
#### 边获取边处理还是一次性获取完再处理？  
因为 Spark 不要求 Shuffle 后的数据全局有序，因此没必要等到全部数据 shuffle 完成后再处理，所以是边 fetch 边处理。
#### 获取来的数据存放到哪里？  
刚获取来的 FileSegment 存放在 softBuffer 缓冲区，经过处理后的数据放在内存 + 磁盘上。
内存使用的是AppendOnlyMap ，类似 Java 的HashMap，内存＋磁盘使用的是ExternalAppendOnlyMap，
如果内存空间不足时，ExternalAppendOnlyMap可以将 records 进行 sort 后 spill（溢出）到磁盘上，等到需要它们的时候再进行归并
#### 怎么获得数据的存放位置？
通过请求 Driver 端的 MapOutputTrackerMaster 询问 ShuffleMapTask 输出的数据位置。

## 触发shuffle的操作 宽依赖触发shuffle action操作划分job 宽依赖划分stage stage是task的集合
![图片](/static/img/v2-6c5382709dc907e1c469d73b12bfbde7_r.jpg)

#### 联系邮箱 xxx_xxx@aliyun.com

