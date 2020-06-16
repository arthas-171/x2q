# hive sql转换为MapReduce过程
### [go back](/hive.md)      
### [go home](../README.md)    
 
## hive sql 转换为 MapReduce过程
+ antlr 定义sql语法规则,完成sql词法,语法解析,将sql转换为抽象语法树AST tree
+ 遍历 AST tree,抽象出查询的基本单元 查询块queryBlock
+ 遍历 queryBlock,翻译成执行操作树 operatorTree
+ 逻辑层优化器进行OperatorTree优化,合并不需要的reduceSinkOperator(合并操作),减少shuffle(遍历清洗)数据量
+ 遍历operatorTree ,翻译成MapReduce任务
+ 物理层优化器进行MapReduce任务的转化,生成最终执行计划
## 一个复杂的hive sql 可能会转化成 多个 MapReduce任务执行
HiveSql  
->AST tree(抽象语法树)  
->query block(查询块)  
->operation tree(执行操作树)  
->逻辑层优化执行操作树 减少重复的合并 减少不必要的shuffle(混洗)  
->new operation tree(新的执行逻辑树)  
->MapReduce task->进行物理层的优化  
->new MapReduce task  



#### 联系邮箱 xxx_xxx@aliyun.com

