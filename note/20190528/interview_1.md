### Thread sleep方法和wait方法之间的区别
    sleep()使当前线程进入停滞状态（阻塞当前线程），让出cpu的使用，目的是不让当前线程独占当前进程的cpu资源，以保留一定
    时间给其他线程执行的机会。
    sleep()是Thread类的Static方法，它不能改变对象锁机制，所以在一个Synchronized代码块中调用sleep(),虽然线程休眠了，但是
    并不会释放锁。
    在sleep()休眠时间到期后，该线程不一定会立即执行，因为其他线程可能正在运行，除非此线程具有更高的优先级。
    
    wait()方法，是Object类里的方法；当一个线程执行到wait()方法时候，它就进入到和该对象相关的线程等待池中，同时会释放掉
    对象锁的占有权，wait(long timeout)会在超时时间过后重新返回对象锁的占有权。
    wait()需要使用notify()、notifyAll()或者指定超时时间来唤醒当前等待池中的线程。
    wait()必须在Synchronized代码块中调用，否则会在program runtime时扔出”java.lang.IllegalMonitorStateException“异常。