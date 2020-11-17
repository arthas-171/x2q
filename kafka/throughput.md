# kafka 高吞吐
### [go back](/x2q/kafka/kafka)      
### [go home](/x2q)      
   
## kafka 如何实现高吞吐

### 分区分段+索引
   首先从设计层面,kafka是标准的分布式程序,采用分布系统分区partition分桶(kafka应该叫分段segment)的设计思想  
topic是个虚拟的概念,一个主题(topic)会被分成多个partition分别储存在不同的broker中,每个broker是一台独立的机器  
partition本质上是每个机器上一个文件夹,partition内部有分为segment(段),每个段是一个文件,每次读写数据其实是操作
这个文件,同时kafka又为segment建立了索引,index(偏移量索引文件)和timeIndex(消息时间戳索引文件),进一步提升效率  
   这种分区分段的思想是高效吞吐的前提,否则细节上再优化单节点也是无法高吞吐的
    ![图片](/static/img/get.jpg)  
    ![图片](/static/img/get1.jpg)  
    ![图片](/static/img/get2.jpg)  
### 磁盘顺序读写
    操作系统的磁盘读写分成两种,一种是随机读写,一种是顺序读写,顺序读写效率是随机的三倍
![图片](/static/img/20190928014327593.jpg)  
    上面说了,kafka的数据本质上是存储在segment中的,segment本质上是磁盘文件,kafka采用顺序写的方式每次追加数据到文件  
结尾,这样效率很高,缺点是不能修改,不支持删除,这样写入的效率问题解决了
    关于读取,每个consumer(消费者)都会有自己的offset,offset记录的是消费每个partition的位置,下次消费的时候继续  
从这个位置往下读取,这样读取也是顺序的
  
### page cache
    数据不可能直接从网络就到磁盘肯定要经过内存,kafka不利用jvm的内存处理数据,而是直接利用操作系统的page cache   
    这样有两个好处
+ 不用考虑jvm GC的问题,不收GC限制
+ 避免object消耗,jvm中object对象其实比数据本身占用内存要大,因为它肯定不能直接是数据本身,还要包含一些其他东西比如引用关系之类的
相比于使用JVM或in-memory cache等数据结构，利用操作系统的Page Cache更加简单可靠。首先，操作系统层面的缓存利用率会更高，因为存
储的都是紧凑的字节结构而不是独立的对象。其次，操作系统本身也对于Page Cache做了大量优化，提供了 write-behind、read-ahead以及
flush等多种机制。再者，即使服务进程重启，系统缓存依然不会消失，避免了in-process cache重建缓存的过程
### 零拷贝
 零拷贝是一个linux技术，Linux通过sendfile方法来实现，将从数据从文件发送到网络的操作的copy（拷贝操作比较占用cpu）
 由4次降低到2次，数据先从文件拷贝到page cache，再由page cache直接使用sendfile方式到socket缓冲区，再由一次copy
 操作到NIC缓冲区，这样是消费端在消费数据的时候效率高，我们会观察到即使大量消费，磁盘的io也不会特别高   
 NIC（network interface card）网络接口卡
    ![图片](/static/img/get2.jpg)     
如果没有"零拷贝"，传输过程会经过用户空间，零拷贝主要是避免了 数据在内核空间和用户空间的拷贝
++ 操作系统将数据从磁盘上读入到内核空间的读缓冲区中。
++ 应用程序（也就是Kafka）从内核空间的读缓冲区将数据拷贝到用户空间的缓冲区中。
++ 应用程序将数据从用户空间的缓冲区再写回到内核空间的socket缓冲区中。
++ 操作系统将socket缓冲区中的数据拷贝到NIC缓冲区中，然后通过网络发送给客户端   
### 批量读写
    这些即使代码层面的优化，不是底层技术，  
    可以是指批次，每次发送一批数据而不是单条，这样效率肯定比单条高
    batch.size=16384(Bytes) producer批量发送的基本单位，默认是 16384 Bytes 
### 批量压缩
     网络传输消耗的网络io，压缩消耗的是cpu，kafka提供对批次数据的压缩，消耗一部cpu能力，减少网络传输的字节数
     解压交给客户端去消耗客户端的cpu完成，支持Gzip和Snappy压缩协议
## 总结
Kafka速度的秘诀在于，它把所有的消息都变成一个批量的文件，并且进行合理的批量压缩，减少网络IO损耗，
通过mmap提高I/O速度，写入数据的时候由于单个Partion是末尾添加所以速度最优；读取数据的时候配合
sendfile直接暴力输出

#### 联系邮箱 xxx_xxx@aliyun.com