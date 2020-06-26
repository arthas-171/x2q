# java 关键字 final,static,volatile,synchronize
### [go back](/java.md)      
### [go home](../README.md)     
## 关键字 final和static
### final (final是终态的意思)关键字修饰的对象,编译时不能被改变
+ 修饰变量不能被改变重新赋值,相当于一个常量
+ 修饰方法不能被重写,子类继承的时候不能被重写
+ 修饰类不能被继承
+ 修饰对象的引用,该引用永远不变但是对象本身可以变,不受影响  

### static (静态的全局的)
+ 只能修饰类里面的属性和方法,还有静态代码块,但是java里面没有全局变量的意思
+ 被static修饰属性和方法和类的实例无关,也是就是说 Class A的静态属性,无论Class A 被new了 多少个 object a,
该属性变量在内存中永远是同一个,并且在Class A被实例化之前就可以访问,因为static 属性并不依赖于对象是否被实例化,
所有实例共享这一个静态属性.

## 关键字 volatile和synchronized
### volatile 本意是 "不稳定的,容易变化的",
他只能用来修饰变量,用他修饰的变量如果发生改变将会通知其他从内存中拷贝了该变量的线程变量值失效,
+ 对象的内存可见性,线程访问变量的时候不是直接访问,而是copy一份到自己的私有内存中,
如果读取直接读取私有内存中的变量值,如果修改先修改私有内存中的变量值.然后同步到jvm内存中,
volatile会保证如果内存中的变量发生了改变会通知其他线程私有内存中的变量失效,需要重新拷贝,从而达到变量的线程安全
+ 执行顺序的不可重排序,jvm会对java代码执行顺序在不影响逻辑的情况下进行优化,但是不会考虑多线程的问题,由此可能带来问题,
volatile关键字会禁止jvm对java代码执行顺序的重排序
### synchronized 本意是 "同步的",也就是说同一时间只能有一个线程运行synchronized修饰的代码
同一时刻最多只有一个线程执行synchronized修饰的代码,从而保证线程并发安全
按照锁的对象可以分为**对象锁** 和**类锁**
按照在代码块中的位置可以分为**方法形式** 和**代码块形式**    
+ 对象锁:如果直接修饰方法那么锁定的this(当前对象),如果是修饰代码块的形式那么括号里面传入的是什么锁定的就是什么可以是this也可以是指定的对象实例
+ 类锁:是锁定的当前类的类对象或者是指定类的类对象,本质上Class对象(类对象)的对象锁,好jb绕口,类对象只有一个所以不同线程访问都会是同步的,注意就算是不同的实例
类锁也只有一个,但是普通的对象锁会是多个锁,如果想要全局同步的话 应该选用类锁,如果是单个对象实例的话可以用对象锁    
**Class对象和实例对象**，实例对象是类的实例，通常是通过new关键字构建的可以创建多个。Class对象是JVM生成用来保存对象的类的信息的,一个类只有一个类对象
                        
##### 先演示下对象锁
                                                      
```java
package com.example.demo.lock;

import java.util.Date;

/**
 * @Auther: huowang
 * @Date: 11:18:36 2020/6/23
 * @DES: 一个简单的线程
 * @Modified By:
 */
public class TestSych implements Runnable{



    @Override
    public void run() {
        // 对象锁当前对象是this 第一个线程执行完,第二个线程才能执行,注意这里 是 thread-1先执行还是 thread-0先执行是随机的 看谁先拿到锁
        method1();
//        Thread-0:代码块方式的的对象锁,开始:Wed Jun 24 21:07:12 CST 2020
//        Thread-0:代码块方式的的对象锁,结束:Wed Jun 24 21:07:17 CST 2020
//        Thread-1:代码块方式的的对象锁,开始:Wed Jun 24 21:07:17 CST 2020
//        Thread-1:代码块方式的的对象锁,结束:Wed Jun 24 21:07:22 CST 2020
        // 对象锁 指定对象都是 lock1 第一个先全部执行完两个锁 第二个才能执行
        method2();
//        Thread-1:AAA代码块方式的的对象锁,开始:Wed Jun 24 21:08:00 CST 2020
//        Thread-1:AAA代码块方式的的对象锁,结束:Wed Jun 24 21:08:05 CST 2020
//        Thread-1:BBB代码块方式的的对象锁,开始:Wed Jun 24 21:08:05 CST 2020
//        Thread-1:BBB代码块方式的的对象锁,结束:Wed Jun 24 21:08:10 CST 2020
//        Thread-0:AAA代码块方式的的对象锁,开始:Wed Jun 24 21:08:10 CST 2020
//        Thread-0:AAA代码块方式的的对象锁,结束:Wed Jun 24 21:08:15 CST 2020
//        Thread-0:BBB代码块方式的的对象锁,开始:Wed Jun 24 21:08:15 CST 2020
//        Thread-0:BBB代码块方式的的对象锁,结束:Wed Jun 24 21:08:20 CST 2020
        // 对象锁 两个锁指定对象分别是 lock1和loc2 第一个执行完lock1 第二个就立刻能拿到lock1 
        method3();
//        Thread-1:AAA代码块方式的的对象锁,开始:Wed Jun 24 21:08:58 CST 2020
//        Thread-1:AAA代码块方式的的对象锁,结束:Wed Jun 24 21:09:03 CST 2020
//        Thread-1:BBB代码块方式的的对象锁,开始:Wed Jun 24 21:09:03 CST 2020
//        Thread-0:AAA代码块方式的的对象锁,开始:Wed Jun 24 21:09:03 CST 2020
//        Thread-1:BBB代码块方式的的对象锁,结束:Wed Jun 24 21:09:08 CST 2020
//        Thread-0:AAA代码块方式的的对象锁,结束:Wed Jun 24 21:09:08 CST 2020
//        Thread-0:BBB代码块方式的的对象锁,开始:Wed Jun 24 21:09:08 CST 2020
//        Thread-0:BBB代码块方式的的对象锁,结束:Wed Jun 24 21:09:13 CST 2020
        // 修饰方法的对象锁  第一个执行完第二个执行
        method4();
//        Thread-0:修饰方法的对象锁的的对象锁,开始:Wed Jun 24 21:11:11 CST 2020
//        Thread-0:修饰方法的对象锁的的对象锁,结束:Wed Jun 24 21:11:16 CST 2020
//        Thread-1:修饰方法的对象锁的的对象锁,开始:Wed Jun 24 21:11:16 CST 2020
//        Thread-1:修饰方法的对象锁的的对象锁,结束:Wed Jun 24 21:11:21 CST 2020
    }

    /**
     * 代码块方式的 对象锁 锁定的是当前对象this
     */
    public  void  method1(){
        synchronized (this){
            System.out.println(Thread.currentThread().getName()+":代码块方式的的对象锁,开始:"+new Date(System.currentTimeMillis()));
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+":代码块方式的的对象锁,结束:"+new Date(System.currentTimeMillis()));
        }
    }

    Object lock1= new Object();
    /**
     * 代码块方式的 对象锁 锁定的是 指定对象 lock1 两个锁都是lock1
     */
    public  void  method2(){

        synchronized (lock1){
            System.out.println(Thread.currentThread().getName()+":AAA代码块方式的的对象锁,开始:"+new Date(System.currentTimeMillis()));
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+":AAA代码块方式的的对象锁,结束:"+new Date(System.currentTimeMillis()));
        }

        synchronized (lock1){
            System.out.println(Thread.currentThread().getName()+":BBB代码块方式的的对象锁,开始:"+new Date(System.currentTimeMillis()));
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+":BBB代码块方式的的对象锁,结束:"+new Date(System.currentTimeMillis()));
        }
    }
    Object lock2= new Object();
    /**
     * 代码块方式的 对象锁 两个锁指定的对象分别是lock1和lock2
     */
    public  void  method3(){
        synchronized (lock1){
            System.out.println(Thread.currentThread().getName()+":AAA代码块方式的的对象锁,开始:"+new Date(System.currentTimeMillis()));
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+":AAA代码块方式的的对象锁,结束:"+new Date(System.currentTimeMillis()));
        }

        synchronized (lock2){
            System.out.println(Thread.currentThread().getName()+":BBB代码块方式的的对象锁,开始:"+new Date(System.currentTimeMillis()));
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+":BBB代码块方式的的对象锁,结束:"+new Date(System.currentTimeMillis()));
        }
    }


    /**
     * 修饰方法的对象锁 锁定的是当前对象this
     */
    public synchronized void  method4(){
            System.out.println(Thread.currentThread().getName()+":修饰方法的对象锁的的对象锁,开始:"+new Date(System.currentTimeMillis()));
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+":修饰方法的对象锁的的对象锁,结束:"+new Date(System.currentTimeMillis()));
    }

    public static void main(String[] args) throws InterruptedException {
        // 注意这里一定是要一个实例 如果是两个实例 对象锁将失去同步性
        TestSych instance = new TestSych();
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
        // join是为了让main线程等待 两个线程结束之后在结束
        t1.join();
        t2.join();
        System.out.println("main 结束了");
    }

}


```    
                       
#### 演示类锁 
                                           
                                           
```java
package com.example.demo.lock;

import java.util.Date;

/**
 * @Auther: huowang
 * @Date: 11:18:36 2020/6/23
 * @DES: 一个简单的线程
 * @Modified By:
 */
public class TestSych1 implements Runnable{



    @Override
    public void run() {
        // 类锁 多个实例 线程同步执行
        method1();
//        Thread-1:修饰方法的类锁,开始:Wed Jun 24 21:49:34 CST 2020
//        Thread-1:修饰方法的类锁,结束:Wed Jun 24 21:49:39 CST 2020
//        Thread-0:修饰方法的类锁,开始:Wed Jun 24 21:49:39 CST 2020
//        Thread-0:修饰方法的类锁,结束:Wed Jun 24 21:49:44 CST 2020
        // 类锁 多个实例 线程同步执行
        method2();
//        Thread-1:代码块方式的的对象锁,开始:Wed Jun 24 21:50:06 CST 2020
//        Thread-1:代码块方式的的对象锁,结束:Wed Jun 24 21:50:11 CST 2020
//        Thread-0:代码块方式的的对象锁,开始:Wed Jun 24 21:50:11 CST 2020
//        Thread-0:代码块方式的的对象锁,结束:Wed Jun 24 21:50:16 CST 2020
        // 对象锁 多个实例 不能保证同步执行
        method3();
//        Thread-0:代码块方式的的对象锁,开始:Wed Jun 24 21:51:23 CST 2020
//        Thread-1:代码块方式的的对象锁,开始:Wed Jun 24 21:51:23 CST 2020
//        Thread-1:代码块方式的的对象锁,结束:Wed Jun 24 21:51:28 CST 2020
//        Thread-0:代码块方式的的对象锁,结束:Wed Jun 24 21:51:28 CST 2020
    }

    /**
     * 修饰方法的类锁 这个方法需要是静态的 静态的就是全局唯一的
     * 如果去掉 static关键字就是 一个对象锁 然后如果弄两个对象实例那么这个方法是不能做到同步到(也就是会两个线程同时操作)
     */
    public synchronized static void  method1(){
            System.out.println(Thread.currentThread().getName()+":修饰方法的类锁,开始:"+new Date(System.currentTimeMillis()));
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+":修饰方法的类锁,结束:"+new Date(System.currentTimeMillis()));
    }


    /**
     * 类锁 修饰代码块的形式  注意括号里面传入的是  *.class 类对象
     */
    public  void  method2(){
        synchronized (TestSych1.class){
            System.out.println(Thread.currentThread().getName()+":代码块方式的的对象锁,开始:"+new Date(System.currentTimeMillis()));
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+":代码块方式的的对象锁,结束:"+new Date(System.currentTimeMillis()));
        }
    }

    Object lock = new Object();
    /**
     * 对象锁 修饰代码块的形式  如果这里传入的是 this 那么多个对象实例就不能保证同步 传入自定义的lock也是一样的效果
     */
    public  void  method3(){
        synchronized (lock){
            System.out.println(Thread.currentThread().getName()+":代码块方式的的对象锁,开始:"+new Date(System.currentTimeMillis()));
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+":代码块方式的的对象锁,结束:"+new Date(System.currentTimeMillis()));
        }
    }
    public static void main(String[] args) throws InterruptedException {
        // 注意这里创建了两个实例
        TestSych1 instance1 = new TestSych1();
        TestSych1 instance2 = new TestSych1();
        Thread t1 = new Thread(instance1);
        Thread t2 = new Thread(instance2);
        t1.start();;
        t2.start();
        t1.join();
        t2.join();
    }

}

```                                                                           
     
                                     
参考连接
[参考连接](https://www.imooc.com/video/18478)                 
#### 联系邮箱 xxx_xxx@aliyun.com

