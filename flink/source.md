#  flink 常用Connector
### [go back](/x2q/flink/flink)      
### [go home](/x2q)       

## MySQL CDC Connector
用于监听mysql表数据的变化
```
CREATE TABLE orders (
     order_id INT,
     order_date TIMESTAMP(0),
     customer_name STRING,
     price DECIMAL(10, 5),
     product_id INT,
     order_status BOOLEAN,
     PRIMARY KEY(order_id) NOT ENFORCED
     ) WITH (
     'connector' = 'mysql-cdc',
     'hostname' = 'localhost',
     'port' = '3306',
     'username' = 'root',
     'password' = '123456',
     'database-name' = 'mydb',
     'table-name' = 'orders');
```

配置选项scan.startup.mode指定MySQL CDC Consumer的启动模式。有效的枚举是：
+ initial（默认）：首次启动时对监控的数据库表进行初始快照，并继续读取最新的binlog。
+ earliest-offset：跳过快照阶段，从最早可访问的 binlog 偏移量开始读取 binlog 事件。
+ latest-offset：首次启动时不会对受监控的数据库表执行快照，仅从 binlog 末尾读取，这意味着只有连接器启动以来的更改。
+ specific-offset：跳过快照阶段并从特定偏移量开始读取binlog事件。可以使用 binlog 文件名和位置指定偏移量，如果在服务器上启用了 GTID，则可以使用 GTID 设置来指定偏移量。
+ timestamp：跳过快照阶段并从特定时间戳开始读取binlog事件。




#### 联系邮箱 xxx_xxx@aliyun.com