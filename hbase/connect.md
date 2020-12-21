# hbase  连接方式和读写
### [go back](/x2q/hbase/hbase)      
### [go home](/x2q)       
## hbase 连接概述
首先要清楚 hbase集群主要的组成部分
+ zookeeper:这个本不属于hbase,只是hbase依赖于它,主要用于获取mate-region的位置,mate-region存储了其他表对应的
region的位置,zk还负责存储集群id和master等信息
+ hbase RegionServer:主要负责管理下属的region,以及这些region的读写,和region的分裂
+ hbase Master :主要负责,建表/删表,为regionServer分配region,hdfs垃圾回收,regionServer之间的负载均衡
由此可见,我们连接hbase和master没有关系,因此我们只需要zk的地址就行,通过zk找到mate-Regina,通过后者找到
具体的表的region,之后和改region所在的regionServer建立联系进行读写 ,过程如下
![图片](/static/img/get2.png)    
-------------------------------------------------------------------------
hbase client与regionServer具体的连接是通过**RpcConnection**对象来实现的,hbase通过一个PoolMap
来存储client到regionServer的连接,本质是一个ConcurrentHashMap,key是封装服务器地址和用户ticket
value是一个RpcConnection对象的资源池,这个池可以通过参数配置大小,每次从里面娶一个连接
+ hbase.client.ipc.pool.type:指定资源池类型 Reusable/RoundRobin/ThreadLocal
+ hbase.client.ipc.pool.size :资源池大小 默认1
## 结论
所以我们不需要单独创建线程池去管理和hbase的连接,因为本质上它自己已经有池化的连接了,一个**进程**
建立一个连接Connection就行,
## 示例代码

## 额外说明
+ hbase 2.0之前对底层的连接是基于原生的socket,2.0之后是基于netty框架
+ hbase shell 命令行,不能显示的指定连接地址,只能通过 hbase-site.xml里面配置的地址进行连接                

#### 联系邮箱 xxx_xxx@aliyun.com