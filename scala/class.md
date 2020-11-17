# class,object,trait,case class
### [go back](/scala.md)      
### [go home](../README.md)     
  
## class
------------------------------------------
在scala中，类名可以和对象名为同一个名字，该对象称为该类的伴生对象，类和伴生对象可以相互访问他们的私有属性
，但是他们必须在同一个源文件内。类只会被编译，不能直接被执行，类的申明和主构造器在一起被申明，在一个类中，
主构造器只有一个,所以必须在内部申明主构造器或者是其他申明主构造器的辅构造器，主构造器会执行类定义中的所有语句。
scala对每个字段都会提供getter和setter方法，同时也可以显示的申明，但是针对val类型，只提供getter方法，
默认情况下，字段为公有类型，可以在setter方法中增加限制条件来限定变量的变化范围，在scala中方法可以访问改类所有对象的私有字段

## case class (case 关键字也可以用于其他地方)
--------------------------------------------
case class 最大的特点是支持模式匹配,其他一些特性如下
+ 初始化的时候可以不用new，当然你也可以加上，普通类一定需要加new；
+ toString的实现更漂亮；
+ 默认实现了equals 和hashCode；
+ 默认是可以序列化的，也就是实现了Serializable ；
+ 自动从scala.Product中继承一些函数;
+ case class构造函数的参数是public级别的，我们可以直接访问；
+ 备注 模式匹配就是关键字 case,不管在 case class中有,比较常见的还有 match
                                               
                                               
```scala
 def matchTest(x: Int): String = x match {
      case 1 => "one"
      case 2 => "two"
      case _ => "many"
   }
```                                                
## object
--------------------------------------------
在scala中没有静态方法和静态字段，所以在scala中可以用object来实现这些功能，直接用对象名调用的方法都是采用这种实现方式，   
例如Array.toString。对象的构造器在第一次使用的时候会被调用，如果一个对象从未被使用，那么他的构造器也不会被执行；   
对象本质上拥有类（scala中）的所有特性，除此之外，object还可以一扩展类以及一个或者多个特质：例如，  
abstract class ClassName（val parameter）{}  
object Test extends ClassName(val parameter){}  
参考[java static](../java/keyWord.md)   
** 注意：object不能提供构造器参数，也就是说object必须是无参的 **

## trait(相当于java 接口 interface)
--------------------------------------------
在java中可以通过interface实现多重继承，在Scala中可以通过特征（trait）实现多重继承，不过与java不同的是，
它可以定义自己的属性和实现方法体，在没有自己的实现方法体时可以认为它时java interface是等价的，
在Scala中也是一般只能继承一个父类，可以通过多个with进行多重继承。

#### 联系邮箱 xxx_xxx@aliyun.com