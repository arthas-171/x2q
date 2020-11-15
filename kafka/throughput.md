# kafka 高吞吐
### [go back](/x2q/kafka/kafka)      
### [go home](/x2q)      
   
## kafka 如何实现高吞吐

### 分区分段+索引
   首先从设计层面,kafka是标准的分布式程序,采用分布系统分区partition分桶(kafka应该叫分段segment)的设计思想  
topic是个虚拟的概念,一个主题(topic)会被分成多个partition分别储存在不同的broker中,每个broker是一台独立的机器  
partition本质上是每个机器上一个文件夹,partition内部有分为segment(段),每个段是一个文件,每次读写数据其实是操作
这个文件,同时kafka又为segment建立了索引,index,进一步提升效率  
   这种分区分段的思想是高效吞吐的前提,否则细节上再优化单节点也是无法高吞吐的
    todo 补上图
### 磁盘顺序读写
    操作系统的磁盘读写分成两种,一种是随机读写,一种是顺序读写,顺序读写效率是随机的三倍
![图片](/static/img/20190928014327593.jpg)  
    上面说了,kafka的数据本质上是存储在segment中的,segment本质上是磁盘文件,kafka采用顺序写的方式每次追加数据到文件  
结尾,这样效率很高,缺点是不能修改,不支持删除,这样写入的效率问题解决了
    关于读取,每个consumer(消费者)都会自己记录自己的offset,offset记录的是消费每个partition的位置,下次消费的时候继续  
从这个位置往下读取,这样读取也是顺序的
    todo 补上offset的图
### page cache
    数据不可能直接从网络就到磁盘肯定要经过内存,kafka不利用jvm的内存处理数据,而是直接利用操作系统的page cache   
    这样有两个好处
+ 不用考虑jvm GC的问题,不收GC限制
+ 避免object消耗,jvm中object对象其实比数据本身占用内存要大,因为它肯定不能直接是数据本身,还要包含一些其他东西比如引用关系之类的
相比于使用JVM或in-memory cache等数据结构，利用操作系统的Page Cache更加简单可靠。首先，操作系统层面的缓存利用率会更高，因为存
储的都是紧凑的字节结构而不是独立的对象。其次，操作系统本身也对于Page Cache做了大量优化，提供了 write-behind、read-ahead以及
flush等多种机制。再者，即使服务进程重启，系统缓存依然不会消失，避免了in-process cache重建缓存的过程
### 零拷贝

### 批量读写

### 批量压缩


#### 联系邮箱 xxx_xxx@aliyun.com