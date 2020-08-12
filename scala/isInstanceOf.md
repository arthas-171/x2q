Scala中isInstanceOf 和 asInstanceOf
### [go back](/scala.md)      
### [go home](../README.md)     
  
## Scala中 isInstanceOf
------------------------------------------
isInstanceOf 翻译一下就是 是否xx的实例 例如, test.isInstanceOf[Test],test是否是Test Class的实例

## Scala中 asInstanceOf
--------------------------------------------
asInstanceOf 就是将对象装换为指定类型 例如 test.asInstanceOf[Test], 最好先用isInstanceOf判断一下再抓换,不然可能会抛出异常.

## lazy 懒加载
lazy val flag:Boolean = true, 定义的时候不加载,真正调用的时候再加载

## TraversableOnce 和 Traversable
TraversableOnce:只能遍历一次的集合
Traversable:可以遍历多次的集合








#### 联系邮箱 xxx_xxx@aliyun.com
