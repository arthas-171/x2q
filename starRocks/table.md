# starRocks table
### [go back](/x2q/starRocks/starRocks)      
### [go home](/x2q)    

## 四种表
### 明细表
默认的表类型， 支持追加新数据，不支持修改历史数据，支持自定义排序键，如果过滤条件包含排序键可以提高查询效率
```roomsql
CREATE TABLE IF NOT EXISTS detail (
event_time DATETIME NOT NULL COMMENT "datetime of event",
event_type INT NOT NULL COMMENT "type of event",
user_id INT COMMENT "id of user",
device_code INT COMMENT "device code",
channel INT COMMENT ""
)
DUPLICATE KEY(event_time, event_type) --制定排序列
DISTRIBUTED BY HASH(user_id)
PROPERTIES (
"replication_num" = "3"
);
```


### 聚合表
建表时，支持定义排序键和指标列，并为指标列指定聚合函数。当多条数据具有相同的排序键时，指标列会进行聚合。
在分析时能提高效率， 例如uv pv，但是无法再查询明细数据， 可以使用bitmap hll等高级的数据结构，在大规模
数据的情况，高效查询指标
```roomsql
CREATE TABLE `page_uv` (
  `page_id` INT NOT NULL COMMENT '页面id',
  `visit_date` datetime NOT NULL COMMENT '访问时间',
  `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT '访问用户id'  ---使用bitmap 计算uv，插入时聚合函数就会提前进行聚合
) ENGINE=OLAP
AGGREGATE KEY(`page_id`, `visit_date`)
DISTRIBUTED BY HASH(`page_id`)
PROPERTIES (
  "replication_num" = "3",
  "storage_format" = "DEFAULT"
);
```
### 更新表
支持定义主键和指标列，查询时返回主键相同的一组数据中的最新数据，新模型可以视为聚合模型的特殊情况，指标列指定的聚合函数为 REPLACE
```roomsql
CREATE TABLE IF NOT EXISTS orders (
    create_time DATE NOT NULL COMMENT "create time of an order",
    order_id BIGINT NOT NULL COMMENT "id of an order",
    order_state INT COMMENT "state of an order",
    total_price BIGINT COMMENT "price of an order"
)
UNIQUE KEY(create_time, order_id)
DISTRIBUTED BY HASH(order_id)
PROPERTIES (
"replication_num" = "3"
); 
```
### 主键表
主键模型支持分别定义主键和排序键。数据导入至主键模型的表时先按照排序键排序后存储。
查询时返回主键相同的一组数据中的最新数据。相对于更新模型，
主键模型在查询时不需要执行聚合操作，并且支持谓词和索引下推，能够在支持实时和频繁更新等场景的同时，提供高效查询。


  
### 参考
https://docs.starrocks.io/zh/docs/table_design/table_types/unique_key_table/
#### 联系邮箱 xxx_xxx@aliyun.com

