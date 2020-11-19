Scala中isInstanceOf 和 asInstanceOf
### [go back](/x2q/scala/scala)      
### [go home](/x2q)       
  
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

## with 关键字 复合类型
可以继承多个特征 或者继承一个类 实现一个接口

## @transient 注解
transient用于标识变量是瞬时的，它不会被持久化,如果给成员变量加@transient注解的话，则相应的成员变量不会被序列化,此时如果进行反序列化的话，对应成员变量为null
一般来说 会给用于"**缓存**"的变量加上此注解







#### 联系邮箱 xxx_xxx@aliyun.com
