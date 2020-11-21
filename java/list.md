# java 中常见数据结构
### [go back](/java.md)      
### [go home](../README.md)     
## list
### ArrayList:不是线程安全的,允许重复数据,本质上是一个动态数据,扩容
+ ArrayList无参构造方法默认容量为10
+ 如果添加的元素超出当前list容量，list容量扩充一半
+ ArrayList最大容量不超过Integer的最大值
### Vector 和 ArrayList 类似,但是线程安全,每个public方法都加了synchronized 关键字,效率很低
## set
### HashSet:线程不安全,基于hashMap实现,扩容机制和hashMap一样 16,0.75
## map
java Map 接口下共有4个实现 分别是
+ hashTable
+ hashMap
+ linkedHashMap
+ treeMap
![图片](/static/img/get12.PNG)   

### hashmMap:线程不安全,默认是链表+数组实现,1.8之后链表长度大于8将转换链表为红黑树
### ConcurrentHashMap:是一个线程安全的map,采用分段锁技术,保证并发性和线程安全
## 数组
## 链表

#### 联系邮箱 xxx_xxx@aliyun.com

