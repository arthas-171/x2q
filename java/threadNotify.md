# java 线程间通讯
### [go back](/java.md)      
### [go home](../README.md)     
## 常见方法 wait()/notify()/notifyAll()/sleep()/yield()/join()
+ wait():所属Object类方法,所有java对象都能条用,让出锁和cpu资源
+ notify():所属Object类方法
+ notifyAll():所属Object类方法
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
#### 联系邮箱 xxx_xxx@aliyun.com

