# hive 常用函数
### [go back](/hive.md)      
### [go home](../README.md)    
 
## 常用函数
+ COALESCE(T v1, T v2,…):返回参数中第一个非null值 ,如果全是null 返回null
+ collect_set/collect_list:列转行 去重/不去重, 配合concat_ws(',',collect_set(userid)) 将集合转为逗号分隔的字符串
+ explode:行转列,对于数组类型,map类型可以行转列,如下,但是只能查询 c1,想要把其他列查出来需要配合 lateral view 

|  c1   | 
|  ----  | 
| [1,2,3]  | 
| [4,5,6] | 
select explode(c1) from t  ;
  
| c1 | 
| ---| 
| 1  | 
| 2  | 
| 3  | 
| 4  | 
| 5  | 
| 6  | 
  
+ lateral view explode: 本质上是将集合拆分成一个表,然后左连接剩下的列的表

|  id   | c1  |
|  ----  | ----  |
| a  | [1,2,3] |
| b  | [4,5,6] |

select id,cc1 from t lateral view explode c1 ct as cc1;

|  id   | c1  |
|  ----  | ----  |
| a  | 1 |
| a  | 2 |
| a  | 3 |
| b  | 4 |
| b  | 5 |
| b  | 6 |






#### 联系邮箱 xxx_xxx@aliyun.com

