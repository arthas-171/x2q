#  flink 时间语义和水位线
### [go back](/x2q/flink/flink)      
### [go home](/x2q)       

## flink 时间语义
### Event time,事件创建时间
+ env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime) 告诉flink要使用何种时间语义
                                                  
````scala
    WatermarkStrategy
      .forBoundedOutOfOrderness[(Long, String)](Duration.ofSeconds(20)) //指定水位线为延迟 20秒
      .withTimestampAssigner(new SerializableTimestampAssigner[(Long, String)] {
        // 提取数据中 第一个long型字段为eventTime 注意这个数据必须是一个毫秒值时间戳
        override def extractTimestamp(element: (Long, String), recordTimestamp: Long): Long = element._1  
      })
````                                                  
                                                  
+ source.assignTimestampsAndWatermarks(WatermarkStrategy)                                                  
### Ingestion time,事件进入flink时间
### Processing time,事件在operation算子处理时的时间,当时运行时机器的时间

## watermark 水位线/水印
就相当于在通勤车的司机,明明到了发车的时间了,但是看车内人很少就在多等几分钟,允许迟到的人也赶上这趟班车  
其实就是允许一定时间内迟到的数据参与该窗口的计算,窗口的计算触发时间也会延迟.
+ watermark 是衡量eventTime进展的机制,可以设置一个延迟
+ watermark 通常用来配合Window处理乱序的数据,等待一会,等乱序的数据都到了(部分到了),重新排序一起处理,也可以不排序
根据业务而定
+ 数据流中的watermark 用于表示timestamp小于watermark的数据都已经到达了,因此Window计算的触发也是由watermark触发的
例如 应该16:05:00 触发窗口计算,其实是watermark水位线高于16:05:00了
+ watermark 让程序编程者自己去权衡数据的延迟和乱序数据的正确性,如果你把watermark设置成延迟一天那么乱序数据
应该肯定是能够按顺序处理,但是延迟太高了
## watermark的实现机理
+ watermark 可以认为是一条插入到数据量中的特殊的时间戳数据
+ watermark 是单向递增的
+ watermark 与数据的时间戳相关
![图片](/static/img/20200804161139.png)  
## watermark 任务之间的传递
每个task有一个自身的task clock任务时钟,task会取各个分区里面最小的一个watermark最为它的watermark传递给下游的task
一个分区的watermark来了时候会与该分区的watermark比较 大于则更新,之后task会比较全部分区的watermark,如果选取最小的
watermark作为task的watermark,可能新来的并不是最小的   
多个输入流的话以最小的watermark为准触发计算
![图片](/static/img/20200804163136.png)  

## 水印生成策略
+ 周期性生成水印,常用的BoundedOutOfOrdernessGenerator是定期水印生成
+ 特殊标识水印生成
### 特殊标识水印生成(标点符号)水印生成器
这种水印生成器,可以允许我们自己定义生成策略,根据某些特殊的数据的到来 生成水印   
理论上可以给每条数据都打伤水印,但是**注意**!每个水印都会在下游触发一系列操作,
因此flink不建议打过多的水印,这将降低flink的处理效率   
表单符号水印实例如下:
                                                         
                                                         
```scala
class PunctuatedAssigner extends AssignerWithPunctuatedWatermarks[MyEvent] {

    override def onEvent(element: MyEvent, eventTimestamp: Long): Unit = {
        if (event.hasWatermarkMarker()) {
            output.emitWatermark(new Watermark(event.getWatermarkTimestamp()))
        }
    }

    override def onPeriodicEmit(): Unit = {
        // don't need to do anything because we emit in reaction to events above
    }
}
```

### java 代码水印例子
```java
 StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.getCheckpointConfig().setCheckpointInterval(3 * 60 * 1000); // 时间间隔
        env.getCheckpointConfig().setMinPauseBetweenCheckpoints(3 * 60 * 1000); //两次checkpoint之间最小时间间隔
        env.getCheckpointConfig().setCheckpointTimeout(60 * 60 * 1000); //checkpoint的超时时间。即每次checkpoint过程可容忍的最大耗费时间。
        env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.AT_LEAST_ONCE);// 一致性模式
        env.getCheckpointConfig().enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.DELETE_ON_CANCELLATION);
        // checkpoint保存在外部存储系统。参数ExternalizedCheckpointCleanup包含两种模式：DELETE_ON_CANCELLATION（作业cancel的时候自动删除所有checkpoint状态）和RETAIN_ON_CANCELLATION（作业取消的时候保留checkpoint状态）。

        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        env.getConfig().setAutoWatermarkInterval(300);// 默认200毫秒

        StreamTableEnvironment tEnv = StreamTableEnvironment.create(env);

        String sourceSql ="CREATE  TABLE alsc_cdm_dwd_alsc_log_page_base_ri_for_HwEventTime(\n" +
                "    `logid` VARCHAR,\n" +
                "  )\n" +
                "WITH (\n" +
                "    'connector' = 'tt',\n" +
                "    'router' = 'http://logcenter.alibaba-inc.com/clientapi',\n" +
                "    'topic' = 'dwd_alsc_log_page_base_ri',\n" +
                "    'lineDelimiter' = '\\n',\n" +
                "    'fieldDelimiter' = '\\u0001',\n" +
                "    'encoding' = 'utf-8',\n" +
                "    'accessId' = '111111111111',\n" +
                "    'accessKey' = '222222222222'\n" +
                "  );"

                ;



        tEnv.executeSql(sourceSql);
        Table inputTable = tEnv.from("alsc_cdm_dwd_alsc_log_page_base_ri_for_HwEventTime");
        DataStream<Row> inputStream = tEnv.toDataStream(inputTable);

        inputStream.
                filter(row -> Objects.nonNull(row.getField("eleme_user_id")))
                .map(x -> {

                    String pattern = "yyyy-MM-dd HH:mm:ss";
                    DateTimeFormatter formatter = DateTimeFormatter.ofPattern(pattern);
                    LocalDateTime localDateTime = LocalDateTime.parse( x.getField("local_time").toString().substring(0,19),formatter);
                    Long timeStamp = localDateTime.toInstant(ZoneOffset.ofHours(8)).toEpochMilli();


                    LogPage pg = new LogPage();
                    pg.setEleme_user_id(x.getField("eleme_user_id").toString());
                    pg.setLocal_time(x.getField("local_time").toString());
                    pg.setSpm_cnt(x.getField("spm_cnt").toString());
                    pg.setClient_code(x.getField("client_code").toString());
                    pg.setUrl_type(x.getField("url_type").toString());
                    pg.setSt(timeStamp);
                    return pg;
                })
//                .assignTimestampsAndWatermarks(
//                        ////  无乱序的 不设置水印的等待时间
//                        new AscendingTimestampExtractor<LogPage>() {
//                            @Override
//                            public long extractAscendingTimestamp(LogPage element) {
//                                return element.getSt();
//                            }
//                        }
//                )
                .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<LogPage>(Time.seconds(2)) {
                    @Override
                    public long extractTimestamp(LogPage element) {
                        // 注册事件时间的时间戳
                        return element.getSt();
                    }
                })
                .map(x->{
                   return x.toString();
                }).print();

        env.execute();

```


[参考链接 事件时间&水印](https://xie.infoq.cn/article/cbd45915022219829b16dc870)

[参考链接 事件时间&水印](https://xie.infoq.cn/article/cbd45915022219829b16dc870)

参考链接 水印延迟和窗口延迟  https://blog.csdn.net/sinat_37719426/article/details/132452105

### 关于数据一致性
#### 通过checkpoint设置 env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.AT_LEAST_ONCE);
+ at-most-once（最多一次）: 这其实是没有正确性保障的委婉说法——故障发生之后，计数结果可能丢失。
+ at-least-once （至少一次）: 这表示计数结果可能大于正确值，但绝不会小于正确值。也就是说，计数程序在发生故障后可能多算，但是绝不会少算。如果有去重机制和sink的幂等写入可以考虑这种
+ exactly-once （精确一次）: 这指的是系统保证在发生故障后得到的计数结果与正确值一致。恰好处理一次是最严格的保证，也是最难实现的。

#### 联系邮箱 xxx_xxx@aliyun.com