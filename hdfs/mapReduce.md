# MapReduce框架
### [go back](/x2q/hdfs/hdfs)      
### [go home](/x2q)     
##Mapreduce初析
Mapreduce是一个计算框架，输入<k,v>类型的值得到一个计算之后的<k,v>类型的值,重点在于它是分布式运行的,也就是说计算的第一阶段Map会在多个不同节点上同时执行一套计算逻辑,之后再通过combiner shuffle 等阶段提交到一个reduce,reduce是对多个map执行结果的汇总(或者也可以不是汇总而且另一种计算,它的输入值是全部map的输出值)
在运行一个mapreduce计算任务时候，任务过程被分为两个阶段：map阶段和reduce阶段，每个阶段都是用键值对（key/value）作为输入（input）和输出（output）。而程序员要做的就是定义好这两个阶段的函数：map函数和reduce函数。

## 逻辑概念 
### 完整的MapReduce包含5个阶段

+ 输入分片（input split）:在执行计算之前根据数据文件进行分片,每个分片对应一个map任务,输入分片不是输入的具体数据而是记录的位置和分片的长度,分片的大小应该基本上和hdfs block块的大小一致(或者略小于),因为如果一个分片跨域两个数据块,那么很可能这两个数据块不在一个节点,这样就需要数据库夸网络传输到map所在节点进行计算,产生了不必要的网络传出
+ map阶段:map阶段在每个节点本地执行map函数逻辑
+ combiner阶段:combiner阶段可以理解为本地的reduce,会在每个节点本地执行一些汇总操作(或者别的操作),值得注意的是不是所有逻辑都可以通过combiner优化,类似于求最大值最小值这种可以在combiner节点进行,只把本地节点的最大值传递给下一阶段,但是对于求平均值则不可用在combiner阶段进行,因为combiner会被多次执行,多次执行之后平均值将不再准确
+ shuffle阶段 将map的输出转化为reduce的输入的过程就是shuffle,map输出计算结果是向一个环形内存缓冲区输出的默认是100MB,其中有一个阈值默认是80%,如果达到了阈值就会在磁盘上生产一个spill文件,还会有一个守卫线程将这些文件整理成排序的partition分区,reduce会启动拷贝进程,将各个节点上的map任务计算结果拷贝到reduce所在节点,拷贝的过程也涉及到合并和排序
+ reduce阶段,reduce函数会已shuffle阶段(其实现在还属于shuffle阶段)map生成的分区文件作为输入,reduce会启动复制线程(默认是5个)复制这些数据,复制这些数据的时候也会有一个缓冲区,缓冲区被带到阈值之后会将数据写入磁盘文件,之后还会有一个合并文件的过程(这里也可以指定参数 合并因子),最终合并完之后的数据会作为输入传递给reduce reduce在执行自定义的逻辑
  
## yarn 控制MapReduce过程的一些概念：
### client 向yarn提交job,先找到ResourceManager分配资源
+ ResourceManager 开启一个container 在container中运行一个Application Manager；
+ Application Manager 找到一个台nodemanager启动Application master
+ Application master向ApplicationManager 申请运行任务需要的资源
+ Resource Scheduler将资源封装好发给 ApplicationMaster
+ Application master将资源分配到 nodemanager
+ 各个nodemanager得到任务和资源开始执行map task
+ map task执行结束后，开始执行reduce task
+ map task和 reduce task将执行结果反馈给Application master
+ Application master将任务执行的结果反馈pplication manager

#### 联系邮箱 xxx_xxx@aliyun.com