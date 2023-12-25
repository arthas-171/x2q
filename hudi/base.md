# hudi 数据湖
### [go back](/x2q/starRocks/starRocks)      
### [go home](/x2q)    
## 概述
### 核心概念
+ instants：时刻 即时时间 hudi的核心是维护表在不同时刻的数据快照状态
  + commit 提交，一次提交将一批数据原子性的写入表
  + instants time：通常就是一个时间戳
  + instants state：状态
    + requested：表示已经请求但是尚未执行
    + inflight：正在执行
    + completeed：执行完成
+ hudi的数据文件
  + 元数据：.hoodie，版本，提交记录等一系列元数据信息
  + 数据文件：以分区的方式存放，包含基本文件（base file parquet格式）和增量日志文件（log file avro格式）
![图片](/static/img/hudi01.jpeg)
  + 分区 partition-》文件组 file group-》文件片 file slice-》base file 和 log file




+ 更新：hudi 相较于hive 可以提供准时的表， 支持更新，而不用像hive一样每次完全覆盖， hudi自身维护了key-file
每次更新时会找到key对应的file进行操作
+ 增量查询：减少查询的数据量，只查询某个时间点之后写入的数据

## table 
### cow and mor
+ cow  copy on write
+ mor  merga on read
+ 对于读写：cow 写入延迟大  mor 读取延迟大，但是mor可以配置压缩策略减少读取延迟， 
+ 对于更新：cow 更新的时 i/o成本高，因为cow模式为每批写入创建新的更新的数据文件， mor更新只需要给更新的部分数据写入增量日志
+ 写放大（存储成本）：cow的存储成本高， 因为每次如果有新增会copy旧的文件，因此存在写放大问题，但是可以设置清理策略，
mor模式不存在这个问题
## 查询类型 query type
+ 快照查询 snapshot queries
+ 增量查询 incremental queries
+ 读优化查询 read optimized queries


  

#### 联系邮箱 xxx_xxx@aliyun.com

