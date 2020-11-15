# hive 自定义聚合函数 UDAF
### [go back](/x2q/hive/hive)      
### [go home](/x2q)     
 
## hive 自定义聚合函数 UDAF
**hive的 udaf 是自定义聚合函数 配合group by 使用,接受0行到多行数据 返回一个计算结果值,定义静态内部类 实现UDAFEvaluator的方法 包括入下**  

+ init() 初始化 一般负责初始化内部字段,通常初始化用来存放最终结果的变量
+ iterate() 每次都会对一个新的值进行聚合计算时都调用该方法,一般会根据计算结果更新用来存放最终结果的变量,如果计算正确或者输入值合法就返回true
+ terminatePartial() 这个方法直译过来是"终止部分",部分聚合结果的时候调用该方法 必须返回一个封装了聚合计算当前状态的对象,类似于 MapReduce的combiner 
+ merge() 接受来自 terminatePartial的返回结果,进行合并,hive合并两部分聚合的时候回调用这个方法
+ terminate() 终止方法 返回最终聚合函数结果
## 实例代码 求最大值
                                       
```java
/**
 * UDAF是输入多个数据行，产生一个数据行
 * 用户自定义UDAF必须继承UDFA，必须提供一个实现了UDAFEvaluator接口的内部类
 */
public class MaxiNumber extends UDAF{ 
    public static class MaxiNumberIntUDAFEvaluator implements UDAFEvaluator{ 
        //最终结果 
        private FloatWritable result; 
        //负责初始化计算函数并设置它的内部状态，result是存放最终结果的 
        public void init() { 
            result=null; 
        } 
        //每次对一个新值进行聚集计算都会调用iterate方法 
        public boolean iterate(FloatWritable value) 
        { 
            if(value==null) 
                return false; 
            if(result==null) 
              result=new FloatWritable(value.get()); 
            else
              result.set(Math.max(result.get(), value.get())); 
            return true; 
        } 
                                                                                                                                   
        //Hive需要部分聚集结果的时候会调用该方法 
        //会返回一个封装了聚集计算当前状态的对象 
        public FloatWritable terminatePartial() 
        { 
            return result; 
        } 
        //合并两个部分聚集值会调用这个方法 
        public boolean merge(FloatWritable other) 
        { 
            return iterate(other); 
        } 
        //Hive需要最终聚集结果时候会调用该方法 
        public FloatWritable terminate() 
        { 
            return result; 
        } 
    } 
}

```

                                





#### 联系邮箱 xxx_xxx@aliyun.com

