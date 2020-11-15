#  flink end-to-end 恰好一次性
### [go back](/x2q/flink/flink)      
### [go home](/x2q)       

## sink 端一次性
+ 幂等写入 例如是一个hbase 覆盖key的value值也无所谓,或者输出结果是个数值,不是明细每次输出这个值都是一样的,覆盖前面的值
+ 事务写入 两阶段提交/预写日志
预写日志 WAL write-ahead-log, 缓存一批,等带checkpoint的时候一次性写入sink,相当于弄一批
但是这样有问题,就是这一批可能写到一半的时候就失败了,导致一部分写入了另一部分没有写入,at-least-once
两阶段提交 tow-phase-commit 2PC,外部系统需要支持事务,kafka的有一个默认的实现


#### 联系邮箱 xxx_xxx@aliyun.com