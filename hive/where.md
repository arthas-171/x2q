# sql 中 where和join 生成中间表问题
### [go back](/x2q/hive/hive)      
### [go home](/x2q)       
 
## sql 中 where和join 生成中间表问题
**数据库在通过连接两张或多张表来返回记录时，都会生成一张中间的临时表，然后再将这张临时表返回给用户**
+ join on 的情况,以left join on 为例,它不管on的条件是否为真,都会把左表中全部记录返回到中间表里面,右表中符合on条件的会追加到左表记录后面
+ where 条件的情况,是在临时表生成好之后(例如 from tableA,tableB,中间表则为 tableA记录和tableB记录做笛卡尔积),再根据条件对中间表过滤

## 示例,查询球员tom2个赛季的得分数差
### 表数据,  名字,得分,赛季
![图片](/static/img/ad9237fae884216926189c1e43e58badba9.jpg)  
**错误写法1 这种写法虽然在 on后面限定了只需要 t1.name='tom' 但是结果其他人的数据也一样被查出来了**  
```$xslt
SELECT
	t1. NAME NAME,
	(t1.score - t2.score) score
FROM
	testhw t1
LEFT JOIN testhw t2 ON t1. NAME = t2. NAME
AND t1.season = 1
AND t2.season = 2
AND t1. NAME = 'tom';
```
  
![图片](/static/img/84b0732cea294dd99932ec079d6fa40e494.jpg)  
**正确写法**
```$xslt
SELECT
	t1. NAME NAME,
	(t1.score - t2.score) score
FROM
	testhw t1
LEFT JOIN testhw t2 ON t1. NAME = t2. NAME
AND t2.season = 2
WHERE
	t1. NAME = 'tom'
AND t1.season = 1;
```
  
![图片](/static/img/06041ed6e370084eba3deb310fc115c23d4.jpg)  
**错误写法2**
```$xslt
SELECT
	t1. NAME NAME,
	(t1.score - t2.score) score
FROM
	testhw t1
LEFT JOIN testhw t2 ON t1. NAME = t2. NAME
AND t2.season = 2
WHERE
	t1. NAME = 'tom';
```
![图片](/static/img/ed2738aa2bb4c643259851b68b7bf97b04d.jpg)  
如果where 条件中不加 AND t1.season = 1 会额外计算出一个 "第二赛季-第二赛季的数据"  
**在 on 后面加额外限定的条件的结果对比**  
```$xslt
SELECT
	t1. NAME,
	t1.season,
	t2.season,
	(t1.score - t2.score) score
FROM
	testhw t1
LEFT JOIN testhw t2 ON t1. NAME = t2. NAME
AND t2.season = 2
AND t1.season = 1
WHERE
	t1. NAME = 'tom';
```
![图片](/static/img/0ae3e587c3756249e643882dc5bc8d07761.jpg)  
on 后面的限定条件只会限定 t2表数据 不会对 t1表产生任何影响 例如 on 后面限定了 t1.season=1 但是结果中仍然包含 t1表 season=2 的数据  

hive 版本 2.3.3
#### 联系邮箱 xxx_xxx@aliyun.com

