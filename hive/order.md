# Hive中order by，sort by，distribute by，cluster by的区别
### [go back](/x2q/hive/hive)      
### [go home](/x2q)      
 
+ order by 会对数据进行全局排序 只有一个reduce 保证全局有序,数据规模比较大的时候回耗费很多时间
+ sort by 在数据进入reduce之前完成排序,可以保证局部有序,不能保证全局有序
+ distribute by 是控制map输出如何划分给不同的reduce,比如说我们想把 userId 相同的数据划分到一个reduce里面 就是 distribute by userId,它可以配合sort by 使用 比如为每个用户的消费金额排序 ,distribute by userId sort by pay
+ cluster by 的功能是distribute by 和sort by 的结合 只不过它只支持降序







#### 联系邮箱 xxx_xxx@aliyun.com

