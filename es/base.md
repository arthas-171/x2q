#  es 基础
### [go back](/x2q/es/es)      
### [go home](/x2q)       

## 基础概念
![图片](/static/img/get1.png)  
+ （1）关系型数据库中的数据库（DataBase），等价于ES中的索引（Index）
+ （2）一个数据库下面有N张表（Table），等价于1个索引Index下面有N多类型（Type），
+ （3）一个数据库表（Table）下的数据由多行（ROW）多列（column，属性）组成，等价于1个Type由多个文档（Document）和多Field组成。
+ （4）在一个关系型数据库里面，schema定义了表、每个表的字段，还有表和字段之间的关系。 与之对应的，在ES中：Mapping定义索引下的Type的字段处理规则，即索引如何建立、索引类型、是否保存原始索引JSON文档、是否压缩原始JSON文档、是否需要分词处理、如何进行分词处理等。
+ （5）在数据库中的增insert、删delete、改update、查search操作等价于ES中的增PUT/POST、删Delete、改_update、查GET.

#### 联系邮箱 xxx_xxx@aliyun.com