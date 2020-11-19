# java 线程间通讯
### [go back](/java.md)      
### [go home](../README.md)     
## 常见方法 wait()/notify()/notifyAll()/sleep()/yield()/join()
+ wait():所属Object类方法,所有java对象都能条用,让出锁和cpu资源,必须是锁对象才能调用
+ notify():所属Object类方法,只随机唤醒一个 wait 线程
+ notifyAll():所属Object类方法,唤醒所有 wait 线程
+ sleep():所属Thread类独有方法,会让出cpu资源,但是不让出锁
+ yield():所属Thread类独有方法,可能让出cpu,但是不让出锁,yield会通知其他线程,如果其他线程优先级高于当前线程则能够获得cpu资源
,否则当前线程会继续持有cpu资源
+ join():所属Thread类独有方法,例如 在main方法中(main方法也是一个线程),子线程t调用了,t.join,
那么main方法会等待t线程执行结束在执行

**join示例代码 join方法需要捕获异常**  
![图片](/static/img/get9.PNG)    
![图片](/static/img/get10.PNG)    

**注意**cpu只能串行执行任务,我们看到的并行只是cpu先执行A线程,在执行B线程,在执行A线程,这样   
## 线程的几种状态/生命周期
####  新建(new):就是刚刚创建的时候 Thread t = new MyThread();
#### 就绪(runable):调用了start方法,t.start(),此时只是可以运行状态,随时等待CPU调度执行，获取cpu 的使用权
#### 运行(running):获取到了cpu资源真正开始运行了,线程只能由runable变成running其他状态都不能直接进入running
#### 阻塞(blocked):有三种情况都会发生阻塞
+ 等待阻塞:t.wait(),调用了wait方法几开始等待一定的时间或者被notify等唤醒,jvm会把线程放入waitting queue(等待队列)
+ 同步阻塞:t线程在获取某个对象的锁的时候,如果锁已经被其他线程或者那么他就会同步阻塞,jvm把t放入lock pool(锁池)
+ 其他阻塞:t.sleep(),t中的其他线程调用了join,获取I/O资源等待,这些也会导致阻塞,当sleep时间结束,join线程结束,
I/O资源获得的时候就会runable,注意不是running  
**示例图** 
![图片](/static/img/137084-20190813080541362-1019213130.png)    
#### 死亡(dead):运行结束或者发生异常结束
**备注 start()方法和run()方法**
start() :
它的作用是启动一个新线程。
通过start()方法来启动的新线程，处于就绪（可运行）状态，并没有运行，一旦得到cpu时间片，
就开始执行相应线程的run()方法，这里方法run()称为线程体，它包含了要执行的这个线程的
内容，run方法运行结束，此线程随即终止。start()不能被重复调用。用start方法来启动线程，
真正实现了多线程运行，即无需等待某个线程的run方法体代码执行完毕就直接继续执行下面的
代码。这里无需等待run方法执行完毕，即可继续执行下面的代码，即进行了线程切换。
run() :
run()就和普通的成员方法一样，可以被重复调用。
如果直接调用run方法，并不会启动新线程！程序中依然只有主线程这一个线程，其程序执行路
径还是只有一条，还是要顺序执行，还是要等待run方法体执行完毕后才可继续执行下面的代码，
这样就没有达到多线程的目的。
总结：调用start方法方可启动线程，而run方法只是thread的一个普通方法调用,和你写一个aaa()方法没啥区别，
还是在主线程里执行。


## wait和notify的示例 
### 演示wait 利用对象锁  定义一个 Object lock 作为锁对象
                   
```java
package com.example.demo.lock;

import java.util.Date;

/**
 * @Auther: huowang
 * @Date: 12:21:29 2020/6/26
 * @DES:   线程A 通过构造方法 传入 锁对象lock 然后 synchronized以此lock作为锁
 * @Modified By:
 */
public class ThreadA implements Runnable {

    private Object lock;

    public ThreadA(Object lock) {
        this.lock=lock;
    }

    @Override
    public void run() {
        synchronized (lock){
            System.out.println(Thread.currentThread().getName()+"开始=>"+new Date(System.currentTimeMillis()));
            try {
                lock.wait(10000);// 线程A 会wait 10秒钟 释放锁并且释放cpu
                // lock.wait(100000);// 情况2  线程A wait 100秒
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"结束=>"+new Date(System.currentTimeMillis()));
        }
    }
}

```
                      
```java
package com.example.demo.lock;

import java.util.Date;

/**
 * @Auther: huowang
 * @Date: 12:21:29 2020/6/26
 * @DES: 线程B 通过构造方法 传入 锁对象lock 然后 synchronized以此lock作为锁
 * @Modified By:
 */
public class ThreadB implements Runnable {

    private Object lock;

    public ThreadB(Object lock) {
        this.lock=lock;
    }

    @Override
    public void run() {
        synchronized (lock){
            System.out.println(Thread.currentThread().getName()+"开始=>"+new Date(System.currentTimeMillis()));
            try {
                 // 线程B 不wait 
                Thread.sleep(5000);
                // lock.notify(); //情况2 线程B 睡眠5秒后通知 block状态的线程A 进入runable  
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"结束=>"+new Date(System.currentTimeMillis()));
        }

    }
}

```        
                                            
```java
package com.example.demo.lock;

import java.util.Date;

/**
 * @Auther: huowang
 * @Date: 11:18:36 2020/6/23
 * @DES: 
 * @Modified By:
 */
public class TestWait {


    public static void main(String[] args) throws InterruptedException {
        // 注意这里创建了两个实例
        Object lock = new Object();
        ThreadA instance1 = new ThreadA(lock);
        ThreadB instance2 = new ThreadB(lock);
        Thread t1 = new Thread(instance1,"ThreadA");
        Thread.sleep(1000);// 为了确保ThreadA 能先拿到锁
        Thread t2 = new Thread(instance2,"ThreadB");
        t1.start();
        t2.start();
     // 运行结果如下 线程A 先拿到锁 然后wait释放锁, 然后B拿到锁 B执行完毕 A执行完毕 
     // 因为 A wait了 10秒钟 线程内容是睡眠5秒钟 所以 A 一共运行 15秒钟
    //ThreadA开始=>Fri Jun 26 12:33:20 CST 2020
    //ThreadB开始=>Fri Jun 26 12:33:20 CST 2020
    //ThreadB结束=>Fri Jun 26 12:33:25 CST 2020
    //ThreadA结束=>Fri Jun 26 12:33:35 CST 2020
 
    // 情况2 线程B运行结束之后会主动通知A运行 这样A就不用等待100秒了
  //ThreadA开始=>Fri Jun 26 12:39:19 CST 2020
  //ThreadB开始=>Fri Jun 26 12:39:19 CST 2020
  //ThreadB结束=>Fri Jun 26 12:39:24 CST 2020
  //ThreadA结束=>Fri Jun 26 12:39:29 CST 2020

    }

}

```
                                                                                           
### AB线程交替打印基数偶数
                                   
```java
package com.example.demo.lock;

import java.util.Date;

/**
 * @Auther: huowang
 * @Date: 12:21:29 2020/6/26
 * @DES:
 * @Modified By:
 */
public class ThreadA implements Runnable {

    private TestWait lock;


    public ThreadA(TestWait lock) {
        this.lock=lock;
    }

//    @Override
//    public void run() {
//        synchronized (lock){
//            System.out.println(Thread.currentThread().getName()+"开始=>"+new Date(System.currentTimeMillis()));
//            try {
//                lock.wait(100000);
//                Thread.sleep(5000);
//            } catch (InterruptedException e) {
//                e.printStackTrace();
//            }
//            System.out.println(Thread.currentThread().getName()+"结束=>"+new Date(System.currentTimeMillis()));
//        }
//    }

    @Override
    public void run() {
        synchronized (lock){
            int i=1;
            while (i<100){
                if(lock.flag){
                    System.out.println(Thread.currentThread().getName()+"=>"+i);
                    lock.notify();  // 通知对象锁的其他线程可以获取锁
                    lock.flag=false; // 修改 标记
                    i += 2; // 自增数字
                }else {
                    try {
                        lock.wait();  // 进入wait 阻塞状态
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }

        }
    }


}

```     
                                       
```java
package com.example.demo.lock;

import java.util.Date;

/**
 * @Auther: huowang
 * @Date: 12:21:29 2020/6/26
 * @DES:
 * @Modified By:
 */
public class ThreadB implements Runnable {

    private TestWait lock;

    public ThreadB(TestWait lock) {
        this.lock=lock;
    }

//    @Override
//    public void run() {
//        synchronized (lock){
//            System.out.println(Thread.currentThread().getName()+"开始=>"+new Date(System.currentTimeMillis()));
//            try {
//                Thread.sleep(5000);
//                lock.notify();
//            } catch (InterruptedException e) {
//                e.printStackTrace();
//            }
//            System.out.println(Thread.currentThread().getName()+"结束=>"+new Date(System.currentTimeMillis()));
//        }
//
//    }


    @Override
    public void run() {
        synchronized (lock){
            int i=2;
            while (i<100){
                if(!lock.flag){
                    System.out.println(Thread.currentThread().getName()+"=>"+i);
                    lock.notify();
                    lock.flag=true;
                    i += 2;
                }else {
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }

        }

    }
}

```          
                                
```java
package com.example.demo.lock;

import java.util.Date;

/**
 * @Auther: huowang
 * @Date: 11:18:36 2020/6/23
 * @DES: 一个简单的线程
 * @Modified By:
 */
public class TestWait {

    public   Boolean flag=true;  // 这个标记是为了控制wait 和 notify

    public static void main(String[] args) throws InterruptedException {


        // 注意这里创建了两个实例
        TestWait lock = new TestWait();

        ThreadA instance1 = new ThreadA(lock);
        ThreadB instance2 = new ThreadB(lock);
        Thread t1 = new Thread(instance1,"ThreadA");
        Thread.sleep(1000);
        Thread t2 = new Thread(instance2,"ThreadB");
        t1.start();
        t2.start();
        //ThreadA=>1
        //ThreadB=>2
        //ThreadA=>3
        //ThreadB=>4
        //ThreadA=>5
        //ThreadB=>6
        //ThreadA=>7
        //ThreadB=>8

    }

}

```                                                                                           

#### 联系邮箱 xxx_xxx@aliyun.com

