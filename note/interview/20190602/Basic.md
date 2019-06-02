### [Java并发面试题-基础](http://ifeve.com/javaconcurrency-interview-questions-base/)

### 多线程
> 1. java中有几种方法可以实现一个线程？ 
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
> 2. 如何停止一个正在运行的线程？     
    java中没有提供立即终止线程执行的方法，因为这样会导致共享的数据结构处于不一致的状态。它提供了中断的机制，使一
    个线程可以中断另一个线程的执行。        
    当调用线程的interrupt()方法后，会将该线程的interrupt值更新为true，当线程在执行的时候判断出interrupted=true       
    如果处于非阻塞状态，会清除当前正在执行的工作，然后终止线程。
    如果处于阻塞状态，会抛出InterruptException由线程调用者做操作
> 3. notify()和notifyAll()有什么区别？
    当线程中的对象调用wait()方法后，这个对象所在的线程就会被放入等待线程队列中，
    调用该对象的notify()方法能够将该对象所在的线程从等待线程中移除，重新开始执行。     
    notifyAll()可以唤醒在等待线程队列中的所有线程。
           
> 4. sleep()和wait()有什么区别？
    sleep()是线程方法，它的作用是使当前线程处于就绪的状态，不与其他线程抢占cpu资源。如果一个线程内含有同步锁，
    在调用sleep()方法后，该同步锁并不会释放掉。
    wait()是对象方法，它的作用是使当前线程处于等待线程队列，不参与锁的竞争。如果这个线程内含有同步锁，在调用
    wait()方法后，会释放掉对该同步锁的持有权。
    
> 5. 什么是Daemon线程？它有什么意义？
    当一个线程是Daemon线程后，它不会影响当前线程的运行。常用于任务调度。
> 6. java如何多线程之间的通讯与协作?
    通过多个线程共享同一个变量，并根据共享变量的不同状态，执行不同的任务。

### 锁
> 1. 什么是可重入锁ReentrantLock 
    可重入锁是指同一个线程可以再次持有该线程所持有的锁。
> 2. 当一个线程进入某个对象的synhronized的实例方法后，其它线程是否有可以进入此对象的其它方法？
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
