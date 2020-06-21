# java 线程
### [go back](/java.md)      
### [go home](../README.md)     
## 线程和进程
+ 进程:一个独立的程序就是一个进程,例如spark的一个executor,一个进程至少包含一个线程
+ 进程:cpu调度的最小单位,一个进程里面通常可以包含多个线程
## 创建线程的方式
#### 继承Thread类 
重写run()方法,
#### 实现Runnable接口
实现run()方法,
#### Callable和Future
+ 创建Callable接口的实现类，并实现call()方法，该call()方法将作为线程执行体，并且有返回值。
+ 创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象的call()方法的返回值。
+ 使用FutureTask对象作为Thread对象的target创建并启动新线程。
+ 调用FutureTask对象的get()方法来获得子线程执行结束后的返回值
           
```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;
 
public class CallableThreadTest implements Callable<Integer>
{
 
	public static void main(String[] args)
	{
		CallableThreadTest ctt = new CallableThreadTest();
		FutureTask<Integer> ft = new FutureTask<>(ctt);
		for(int i = 0;i < 100;i++)
		{
			System.out.println(Thread.currentThread().getName()+" 的循环变量i的值"+i);
			if(i==20)
			{
				new Thread(ft,"有返回值的线程").start();
			}
		}
		try
		{
			System.out.println("子线程的返回值："+ft.get());
		} catch (InterruptedException e)
		{
			e.printStackTrace();
		} catch (ExecutionException e)
		{
			e.printStackTrace();
		}
 
	}
 
	@Override
	public Integer call() throws Exception
	{
		int i = 0;
		for(;i<100;i++)
		{
			System.out.println(Thread.currentThread().getName()+" "+i);
		}
		return i;
	}
 
}
```
         
#####因为java中只能单一继承extend 所以使用实现接口的方式更灵活,实现接口后还能继承其他类  
#### 通过线程池创建 (另见线程池)

## 关于线程的异常捕获
首先线程之间是**"独立的代码"**,线程自己的异常不能直接抛出而是应该自己进行try catch捕获,**只要是通过重写 run()** 方法的线程
都不能直接 throw exception.  
例如  
![图片](/static/img/get1.PNG)   
![图片](/static/img/get3.PNG)   
正确写法例如
![图片](/static/img/get2.PNG)   
![图片](/static/img/get4.PNG)   
##### 实现callable接口的
实现 callable接口的 可以跑出 throw 因为本质上他还不是一个线程,他是只一个单纯的接口,如果想用太运行起来,需要放到线程池中提交,
或者通过FutureTask(未来任务)来包装,然后通过Tread线程提交,  
实例
![图片](/static/img/get5.PNG) 
当然不跑出异常 或者使用try catch 也是允许的
## 各种实现方式提交运行实例和多个线程改写同一个静态变量实例
                      
```java
/**
 * @Auther: huowang
 * @Date: 9:43:05 2020/6/20
 * @DES:
 * @Modified By:
 */
public class ExtendThread extends  Thread{

    private  static int cc;

//    @Override
//    public void run() throws  Exception{
//            cc++;
//            System.out.println("cc=>"+cc);
//    }

    @Override
    public void run(){
        try {
            for (int i=0;i<100;i++){
                System.out.println(Thread.currentThread().getName()+":cc=>"+cc++);
            }
        }catch (Exception e){
            System.out.println(e);
        }

    }

}
```
              
```java
/**
 * @Auther: huowang
 * @Date: 12:30:34 2020/6/20
 * @DES:
 * @Modified By:
 */
public class ImplRunable implements Runnable {

    private static int cc=0;

    @Override
    public void run() {
        try {
            for (int i=0;i<100;i++){
                System.out.println(Thread.currentThread().getName()+":cc=>"+cc++);
            }
        }catch (Exception e){
            System.out.println(e);
        }

    }

//    @Override
//    public void run() throws Exception{
//        System.out.println("cc=>"+cc++);
//    }


}
```
               
```java
/**
 * @Auther: huowang
 * @Date: 12:37:26 2020/6/20
 * @DES:
 * @Modified By:
 */
public class ImplCallable implements Callable<String> {

    private static int cc=0;

    /**
     * 返回一个 string类型
     * @return
     * @throws Exception
     */
    @Override
    public String call() throws Exception {
        for (int i=0;i<100;i++){
            cc++;
        }
        //多线程执行这个结果大概率不会是100,而是一个大于100的数 以为cc是静态的 多个线程会交替改写她的值
        return "cc结果:"+cc;
    }
}
```
                   
```java
/**
 * @Auther: huowang
 * @Date: 13:01:30 2020/6/20
 * @DES:
 * @Modified By:
 */
public class TestMain {
    public static void main(String[] args) {

        ImplCallable c1 = new ImplCallable();
        ImplCallable c2 = new ImplCallable();
        FutureTask<String> f1=new FutureTask<>(c1);
        FutureTask<String> f2=new FutureTask<>(c2);
        Thread cc1 = new Thread(f1);
        Thread cc2 = new Thread(f2);
        try {
           cc1.start();
           cc2.start();
           // 大概率两个的输出结果都是大于100的数字
           System.out.println(f1.get());;
           System.out.println(f2.get());;
        } catch (Exception e) {
            e.printStackTrace();
        }


        ImplRunable r1 = new ImplRunable();
        ImplRunable r2 = new ImplRunable();
        Thread rr1 = new Thread(r1);
        Thread rr2 = new Thread(r2);
        // 会随机交替打印结果
                rr1.start();
                rr2.start();

        ExtendThread t1 = new ExtendThread();
        ExtendThread t2 = new ExtendThread();
        // 会随机交替打印结果
        t1.start();
        t2.start();
    }
}

```
#### 联系邮箱 xxx_xxx@aliyun.com

