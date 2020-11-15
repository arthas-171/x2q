# hive join,outer join, semi join详解
### [go back](/x2q/hive/hive)      
### [go home](/x2q)     
 
## hive join,outer join, semi join详解
+ join 最简单 两个表取交集
+ left outer join是以左表驱动，右表不存在的key均赋值为null
+ right outer join是以右表驱动，左表不存在的key均赋值为null
### 此外hive sql不支持 in函数 比如

SELECT a.key, a.value
FROM a
WHERE a.key in (SELECT b.key FROM B);

### 这个可以用 left outer join替换

SELECT a.key, a.value
FROM a LEFT OUTER JOIN b ON (a.key = b.key)
WHERE b.key <> NULL;

### 更高端的写法是  用 semi join

SELECT a.key, a.value
FROM a LEFT SEMI JOIN b on (a.key = b.key);






#### 联系邮箱 xxx_xxx@aliyun.com

