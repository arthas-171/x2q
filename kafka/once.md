# kafka 恰好一次和事务消息
### [go back](/x2q/kafka/kafka)      
### [go home](/x2q)       
  
## 概述
kafka内部的恰好一次,需要保证producer(生产者),consumer(消费者)都是**恰好一次**
+ producer:写入数据不重复,不丢失
+ consumer:读取数据不重复,不丢失

## 在 kafka  0.11.0.0之前 只能做到 至少一次
+ producer 通过设置ack来控制,1(leader接收成功)和all(leader和副本全部接收成功),就是**至少一次**,0(不等待返回消息)就是**至多一次**
+ consumer 通过设置 offset提交策略(enable.auto.commit=true)来控制,如果是true 表示从poll完之后就提交不关心是否真正消费完毕,
就是**至多一次**,如果是false 需要自己去定义提交的逻辑,可以采用同步/异步两种方式,区别在于同步会失败重试   
关于 consumer的自动提交 还有两个参数分别是 
+ auto.commit.interval.ms:如果是自动提交 true,该参数表示自动提交的时间间隔
+ auto.offset.reset:这个是 offset的重置策略,有三种 
    + none:没有策略,topic各分区都存在已提交的offset时，从offset后开始消费；只要有一个分区不存在已提交的offset，则抛出异常
    + earliest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费
    + latest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据
-----------------------------------
这里有两个问题,第一个是0.11之前不能实现恰好一次,第二个是 consumer手动提交需要自己控制逻辑
##  0.11.0.0之后 

### producer引入了 幂等提交和事务提交 
+ 幂等提交是enable.idempotent=true
    + 每个新的Producer在初始化的时候会被分配一个唯一的PID，这个PID对用户是不可见的。
    + Producer发送数据的每个<Topic, Partition>都对应一个从0开始单调递增的Sequence Number,如果等于+1就写入,小于等于就丢弃,如果大于+1就报错
    这样可以保证 单个Producer对于同一个<Topic, Partition>的Exactly Once语义,不能保证同一个Producer一个topic不同的partion幂等
+ 事务提交
    + 对于Producer，需要设置transactional.id属性，设置了transactional.id属性后，enable.idempotence属性会自动设置为true
    + 对于Consumer，需要设置isolation.level = read_committed，这样Consumer只会读取已经提交了事务的消息。
    另外，需要设置enable.auto.commit = false来关闭自动提交Offset功能。
如图开启 事务
 ![图片](/static/img/get6.png)
kafka 通过引入 Transaction Coordinator(事务协调器)来主要负责分配pid，记录事务状态等操作,
  ![图片](/static/img/get7.png)

##consumer端参数
  consumer端 kafka+flink的设置
  其实只要是kafka 0.11及以后的版本,只需要开启flink的checkpoint恰好一次就行,因为flink会在状态中保存对kafka消费的
  offset,如果开启了主动设置的auto.commit.enable = true就会自动失效
                                                                              
                                                                              
```java
public class RestartStrategyDemo {

    public static void main(String[] args) throws Exception {

        /**1.创建流运行环境**/
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        /**请注意此处：**/
        //1.只有开启了CheckPointing,才会有重启策略
        env.enableCheckpointing(5000);
        //2.默认的重启策略是：固定延迟无限重启
        //此处设置重启策略为：出现异常重启3次，隔5秒一次
        env.getConfig().setRestartStrategy(RestartStrategies.fixedDelayRestart(2, Time.seconds(2)));
        //系统异常退出或人为 Cancel 掉，不删除checkpoint数据
        env.getCheckpointConfig().enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);
        //设置Checkpoint模式（与Kafka整合，一定要设置Checkpoint模式为Exactly_Once）
        env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);


        /**2.Source:读取 Kafka 中的消息**/
        //Kafka props
        Properties properties = new Properties();
        //指定Kafka的Broker地址
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.204.210:9092,192.168.204.211:9092,192.168.204.212:9092");
        //指定组ID
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "testGroup");
        //如果没有记录偏移量，第一次从最开始消费
        properties.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        //Kafka的消费者，不自动提交偏移量
        properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");

        FlinkKafkaConsumer<String> kafkaSource = new FlinkKafkaConsumer("testTopic", new SimpleStringSchema(), properties);

        //Checkpoint成功后，还要向Kafka特殊的topic中写偏移量(此处不建议改为false )
        //设置为false后，则不会向特殊topic中写偏移量。
        //你如果禁用CheckPointing，则Flink Kafka Consumer依赖于内部使用的Kafka客户端的自动定期偏移量提交功能。
        // 该偏移量会被记录在 Kafka 中的 _consumer_offsets 这个特殊记录偏移量的 Topic 中。
        //你如果启用CheckPointing，偏移量则会被记录在 StateBackend 中。该方法kafkaSource.setCommitOffsetsOnCheckpoints(boolean);
        // 设置为 ture 时，偏移量会在 StateBackend 和 Kafka 中的 _consumer_offsets Topic 中都会记录一份；设置为 false 时，偏移量只会在 StateBackend 中的 存储一份。
        //kafkaSource.setCommitOffsetsOnCheckpoints(false);
        //通过addSource()方式，创建 Kafka DataStream
        DataStreamSource<String> kafkaDataStream = env.addSource(kafkaSource);

        /**3.Transformation过程,这个过程是隐式的并没有在代码中提现**/
        SingleOutputStreamOperator<Tuple2<String, Integer>> streamOperator = kafkaDataStream.map(str -> Tuple2.of(str, 1)).returns(Types.TUPLE(Types.STRING, Types.INT));

        /**此部分读取Socket数据，只是用来人为出现异常，触发重启策略。验证重启后是否会再次去读之前已读过的数据(Exactly-Once)*/
        /*************** start **************/
        DataStreamSource<String> socketTextStream = env.socketTextStream("localhost", 8888);

        SingleOutputStreamOperator<String> streamOperator1 = socketTextStream.map(new MapFunction<String, String>() {
            @Override
            public String map(String word) throws Exception {
                if ("error".equals(word)) {
                    throw new RuntimeException("Throw Exception");
                }
                return word;
            }
        });
        /************* end **************/

        //对元组 Tuple2 分组求和
        SingleOutputStreamOperator<Tuple2<String, Integer>> sum = streamOperator.keyBy(0).sum(1);

        /**4.Sink过程**/
        sum.print();

        /**5.任务执行**/
        env.execute("RestartStrategyDemo");
    }
}

```

## note
Flink任务重启时，并未指定 savePoint 路径，为什么还能够恢复数据？
+ 如果任务重启时，指定savePoint路径(Checkpoint路径)，它则会从指定的savePoint路径恢复数据
+ 如果不指定 savePoint 路径，任务会从 Kafka 中的_consumer_offsets这个 topic 中，查看有没有相同group.id 的 topic 的偏移量，如果有的话就会接着之前写入的偏移量来读。
              
  ![图片](/static/img/get8.png)                                                                                     
参考:https://blog.csdn.net/lzb348110175/article/details/104363792
#### 联系邮箱 xxx_xxx@aliyun.com