#  flink 采坑记
### [go back](/flink.md)      
### [go home](../README.md)     

# 这是若干篇悲伤的故事
## flink ui 不显示发送的数据 Bytes Sent 0B, Records Sent 0
**背景**,一个简单的程序 从kafka中读取数据,过滤转换后再写入另一个kafka,本来逻辑很简单,但是观察flink ui 发现没有发送任何数据,
起初以为是程序引入的依赖和kafka版本不对应,经过一个下午+一个晚上+一个早上还是没有发现问题所在,之后把程序精简为直接读取A topic写入
B topic,没有任何逻辑加工发现还是 Bytes Sent 0B 如下图
![图片](/static/img/get2.png)  
无比绝望,但是,但是偶然间我想看看是不是写入的topic是否创建成功,就用命令消费了一下目标topic发现已经有数据写入,what fuck!,这时我以为
是flink ui的bug没有显示发送的数据,但是后来程序逻辑丰富之后发现并不是不显示,而是只有 source的接受和sink的输出不显示,经过查文档和google
发现,原来flink ui 无法计算 source的输入和sink的输出因为这两部已经处于**flink之外**了,received和sent只能统计算子之间的数据流转量
如下图
![图片](/static/img/get3.png)  



#### 联系邮箱 xxx_xxx@aliyun.com