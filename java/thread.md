# java 线程
### [go back](/java.md)      
### [go home](../README.md)     
## 线程和进程
+ 进程:一个独立的程序就是一个进程,例如spark的一个executor,一个进程至少包含一个线程
+ 进程:cpu调度的最小单位,一个进程里面通常可以包含多个线程
## 创建线程的方式
#### 继承Thread类 
重写run()方法,调用该线程使用start()方法
#### 实现Runnable接口
实现run()方法,调用使用start() 方法
#### Callable和Future
+ 创建Callable接口的实现类，并实现call()方法，该call()方法将作为线程执行体，并且有返回值。
+ 创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象的call()方法的返回值。
+ 使用FutureTask对象作为Thread对象的target创建并启动新线程。
+ 调用FutureTask对象的get()方法来获得子线程执行结束后的返回值
```

package com.thread;
 
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
因为java中只能单一继承extend 所以使用实现接口的方式更灵活,实现接口后还能继承其他类  
#### 通过线程池创建 (另见线程池)



#### 联系邮箱 xxx_xxx@aliyun.com

