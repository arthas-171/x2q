# java 中常见数据结构&数据类型
### [go back](/java.md)      
### [go home](../README.md)     

## 基本的数据类型
byte、short、int、long、float、double、boolean、char
String 不是基本数据类型，因为它是一个对象。 它是由Java API中的 java.lang.String 类实现的
## java 最基本的数据结构只有两种一种是数组,一种是指针 其他结构都是基于这两种实现的

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
+ hashTable:Hashtable是前朝遗老，它承自Dictionary类，键值都不可为null，并且是线程安全的，
任一时间只有一个线程能写Hashtable,并发性不好,官方已经不推荐使用,如果需要线程安全可以使用ConcurrentHashMap
+ hashMap:线程不安全,默认是链表+数组实现,1.8之后链表长度大于8将转换链表为红黑树,访问速度快,key无序
+ linkedHashMap:LinkedHashMap是HashMap的一个子类，保存了记录的插入顺序，在用Iterator遍历LinkedHashMap时，
先得到的记录肯定是先插入的,线程不安全
+ treeMap:TreeMap实现SortedMap接口，他支持key排序,线程不安全
![图片](/static/img/get12.PNG)   

### hashmMap:线程不安全,默认是链表+数组实现,1.8之后链表长度大于8将转换链表为红黑树
+ 扩容是一个相对很消耗资源的事情,因此如果可以预见数据规模最好初始化的时候指定大概大小
例如**HashMap<String, String> map = new HashMap<>(100);  //初始化大小100**

#### 扩容有两个参数 一个是初始化桶个数,一个是负载因子(0-1)
+ 初始容量，指明初始的桶的个数；相当于桶数组的大小。
+ 装载因子，是一个0-1之间的系数，根据它来确定需要扩容的阈值，默认值是0.75。
当map中包含的Entry的数量大于等于threshold(阈值) = loadFactor(负载因子) * capacity(容量)的时候，
且新建的Entry刚好落在一个非空的桶上，此刻触发扩容机制，将其容量扩大为2倍(围栏计算hash更有利),当size大于等于threshold的时候，
并不一定会触发扩容机制，但是会很可能就触发扩容机制，只要有一个新建的Entry出现哈希冲突，则立刻resize
#### hashMap 执行put方法的过程
+ 先根据 key调用hashCode方法取hash值
+ 根据key的hash值计算它的存储位置
+ 如果该位置已经存有数据后面接一个链表 
### ConcurrentHashMap:是一个线程安全的map,采用分段锁技术,保证并发性和线程安全,
+ concurrentHashMap由Segment数组结构和HashEntry数组结构组成
+ Segment是一种可重入锁（ReentrantLock），HashEntry用于存储键值对数据
+ 一个ConcurrentHashMap包含一个由若干个Segment对象组成的数组，每个Segment对象守护整个散列映射表的若干个桶，
每个桶是由若干个HashEntry对象链接起来的链表，table是一个由HashEntry对象组成的数组，table数组的每一个数组成
员就是散列映射表的一个桶
**1.7 和 1.8 的 区别**
1.7的时候 concurrentHashMap是用segment+hashEntry实现的
1.8之后使用synchronize+cas 和node+Unsafe 实现,优点是粒度更细了

## 数组
## 链表

#### 联系邮箱 xxx_xxx@aliyun.com

