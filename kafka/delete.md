# kafka 日志清理策略
### [go back](/x2q/kafka/kafka)      
### [go home](/x2q)         
## 概述
kafka 有两个对于数据个删除策略,一种是compaction(压缩),一种是delete(删除)
## delete 删除
delete是kafka默认的清理策略,超过一定大小或者一定时间的日志就会被书喊出,对应的broker配置如下
+ log.retention.bytes=-1 -1是无限
+ log.retention.hours=168 168小时
## compaction 压缩
压缩策略,对于包含多个key的topic有效,它会只保留最新的key的值,旧的删除如下图
![图片](/static/img/get5.png)     


#### 联系邮箱 xxx_xxx@aliyun.com