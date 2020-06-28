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


## shuffle 调优参数,感觉基本上没感觉
### 基本上的思路就是 调大缓冲区 较少拉取/写入文件的次数,
### 如果executor内存够大并且cache缓存较小可以增大shuffle read的操作内存
### 确定不需要排序的话 直接指定shuffmanager为bypass模式
#### spark.shuffle.file.buffer
默认值：32k
参数说明：该参数用于设置shuffle write task的BufferedOutputStream的buffer缓冲大小。将数据写到磁盘文件之前，会先写入buffer缓冲中，待缓冲写满之后，才会溢写到磁盘。
调优建议：如果作业可用的内存资源较为充足的话，可以适当增加这个参数的大小（比如64k），从而减少shuffle write过程中溢写磁盘文件的次数，也就可以减少磁盘IO次数，进而提升性能。在实践中发现，合理调节该参数，性能会有1%~5%的提升。
#### spark.reducer.maxSizeInFlight
默认值：48m
参数说明：该参数用于设置shuffle read task的buffer缓冲大小，而这个buffer缓冲决定了每次能够拉取多少数据。
调优建议：如果作业可用的内存资源较为充足的话，可以适当增加这个参数的大小（比如96m），从而减少拉取数据的次数，也就可以减少网络传输的次数，进而提升性能。在实践中发现，合理调节该参数，性能会有1%~5%的提升。
#### spark.shuffle.io.maxRetries
默认值：3
参数说明：shuffle read task从shuffle write task所在节点拉取属于自己的数据时，如果因为网络异常导致拉取失败，是会自动进行重试的。该参数就代表了可以重试的最大次数。如果在指定次数之内拉取还是没有成功，就可能会导致作业执行失败。
调优建议：对于那些包含了特别耗时的shuffle操作的作业，建议增加重试最大次数（比如60次），以避免由于JVM的full gc或者网络不稳定等因素导致的数据拉取失败。在实践中发现，对于针对超大数据量（数十亿~上百亿）的shuffle过程，调节该参数可以大幅度提升稳定性。
#### spark.shuffle.io.retryWait
默认值：5s
参数说明：具体解释同上，该参数代表了每次重试拉取数据的等待间隔，默认是5s。
调优建议：建议加大间隔时长（比如60s），以增加shuffle操作的稳定性。
#### spark.shuffle.memoryFraction
默认值：0.2
参数说明：该参数代表了Executor内存中，分配给shuffle read task进行聚合操作的内存比例，默认是20%。
调优建议：在资源参数调优中讲解过这个参数。如果内存充足，而且很少使用持久化操作，建议调高这个比例，给shuffle read的聚合操作更多内存，以避免由于内存不足导致聚合过程中频繁读写磁盘。在实践中发现，合理调节该参数可以将性能提升10%左右。
#### spark.shuffle.manager
默认值：sort
参数说明：该参数用于设置ShuffleManager的类型。Spark 1.5以后，有三个可选项：hash、sort和tungsten-sort。HashShuffleManager是Spark 1.2以前的默认选项，但是Spark 1.2以及之后的版本默认都是SortShuffleManager了。tungsten-sort与sort类似，但是使用了tungsten计划中的堆外内存管理机制，内存使用效率更高。
调优建议：由于SortShuffleManager默认会对数据进行排序，因此如果你的业务逻辑中需要该排序机制的话，则使用默认的SortShuffleManager就可以；而如果你的业务逻辑不需要对数据进行排序，那么建议参考后面的几个参数调优，通过bypass机制或优化的HashShuffleManager来避免排序操作，同时提供较好的磁盘读写性能。这里要注意的是，tungsten-sort要慎用，因为之前发现了一些相应的bug。
#### spark.shuffle.sort.bypassMergeThreshold
默认值：200
参数说明：当ShuffleManager为SortShuffleManager时，如果shuffle read task的数量小于这个阈值（默认是200），则shuffle write过程中不会进行排序操作，而是直接按照未经优化的HashShuffleManager的方式去写数据，但是最后会将每个task产生的所有临时磁盘文件都合并成一个文件，并会创建单独的索引文件。
调优建议：当你使用SortShuffleManager时，如果的确不需要排序操作，那么建议将这个参数调大一些，大于shuffle read task的数量。那么此时就会自动启用bypass机制，map-side就不会进行排序了，减少了排序的性能开销。但是这种方式下，依然会产生大量的磁盘文件，因此shuffle write性能有待提高。

#### 联系邮箱 xxx_xxx@aliyun.com

