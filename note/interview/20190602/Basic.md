### [Java并发面试题-基础](http://ifeve.com/javaconcurrency-interview-questions-base/)

### 多线程
> ##### 1. java中有几种方法可以实现一个线程？         
   > 有两种方法可以实现一个线程：     
    1. 实现Runnable接口并实现run方法，新建一个Thread对象，并将实现的参数对象作为参数传入。       
    2. 继承Thread类
```$xslt
        public class MyThread implements Runnable{
            @Override
            public void run() {
                System.out.println("Hello world");
            }
        
            @Test
            public void test(){
                new Thread(new MyThread()).start();
                new OtherThread().start();
            }
        
            class OtherThread extends Thread{
                @Override
                public void run(){
                    System.out.println("Hello world");
                }
            }
        }

```         
   
> ##### 2. 如何停止一个正在运行的线程？
   > 有两种方法可以实现一个线程：

> ##### 3. notify()和notifyAll()有什么区别？
> ##### 4. sleep()和wait()有什么区别？
> ##### 5. 什么是Daemon线程？它有什么意义？
> ##### 6. java如何多线程之间的通讯与协作?

### 锁
> 1. 什么是可重入锁ReentrantLock 
> 2. 当一个线程进入某个对象的synhronized的实例方法后，其它线程是否可以进入此对象的其它方法？
> 3. synchronized和java.util.concurrent.locks.Loack的相同点和不同点？
> 4. 乐观锁和悲观锁的理解以及如何实现，有哪些实现方式？

### 并发框架   
> 1. SynchronizedMap和ConcurrentHashMap有什么区别？ 
> 2. CopyOnWriteArrayList可以用于什么应用场景？

### 线程安全
> 1. 什么叫线程安全？servlet是线程安全的吗？ 
> 2. 同步有几种实现方法?
> 3. volatile有什么用？能否用一句话说明下volatile的应用场景？
> 4. 请说明下java的内存模型以及工作流程
> 5. 为什么代码会重排序   
