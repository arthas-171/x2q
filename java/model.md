# java 设计模型
### [go back](/java.md)      
### [go home](../README.md)     
## 单例模式
                        
```java

package com.example.demo.model;

/**
 * @Auther: huowang
 * @Date: 10:52:02 2020/6/27
 * @DES: 
 * @Modified By:
 */
public class SingleMan0 {

    // 不是单例模型 就是一个普通的类 其实构造方法也可以不写 默认就有无参构造
    public SingleMan0() {
    }
}

```
                          
```java
package com.example.demo.model;

/**
 * @Auther: huowang
 * @Date: 10:42:14 2020/6/27
 * @DES:  单例模式1  不加载初始化, 需要的时候再创建 
 * @Modified By:
 */
public class SingleMan1 {
    // 注意这个 singleMan是 静态私有的 外部不能直接访问
    private static SingleMan1 singleMan= null;
    // 类加载的时候并不顺便初始化实例 ,而是需要的时候再初始化
    // 需要注意 这里的类锁(synchronized +static 详情见 java基础->java 关键字),确保所有线程拿到的都是一把锁
    public synchronized static SingleMan1 getOne(){
        if(null==singleMan){
            singleMan=new SingleMan1();
        }
        return singleMan;
    }
}

```                          
                                            
```java
package com.example.demo.model;

/**
 * @Auther: huowang
 * @Date: 10:36:52 2020/6/27
 * @DES:   单例模型 类加载就初始化好 以后再也不变了
 * @Modified By:
 */
public class SingleMan2 {

    // 单例模型 在利用 static 在类加载的时候就初始化好 实例 并且是 final不可变的
    private final static SingleMan2 singleMan= new SingleMan2();

    public SingleMan2 getOne(){
        return singleMan;
    }

}

```                                            
                          
```java
package com.example.demo.model;

/**
 * @Auther: huowang
 * @Date: 10:47:55 2020/6/27
 * @DES:  测试类
 * @Modified By:
 */
public class TestModel  {


    public static void main(String[] args) {


        SingleMan0 singleMan0a = new SingleMan0();
        SingleMan0 singleMan0b = new SingleMan0();
        // == 比较非基础类型 比较的是内存地址
        if (singleMan0a==singleMan0b){
            System.out.println("实例相等");
        }else {
            System.out.println("实例不相等");
        }
        // 输出 实例不相等

        if (SingleMan1.getOne()== SingleMan1.getOne()){
            System.out.println("实例相等");
        }else {
            System.out.println("实例不相等");
        }
         // 输出 实例相等
        SingleMan2 singleMan2a = new SingleMan2();
        SingleMan2 singleMan2b = new SingleMan2();
        if (singleMan2a.getOne()==singleMan2b.getOne()){
            System.out.println("实例相等");
        }else {
            System.out.println("实例不相等");
        }
         // 输出 实例相等
    }
}

```                          


#### 联系邮箱 xxx_xxx@aliyun.com

