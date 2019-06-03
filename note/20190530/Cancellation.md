### Java内存模型Cookbook阅读笔记 参考链接：[并发编程网-JMM Cookbook](http://ifeve.com/cancellation/)

### 取消线程要确保多线程共享的对象的状态一致性，所以线程取消采取一种协作中断的方式。            
    线程在同步块中或IO中阻塞时候，线程在等待synchronized方法或synchronized锁时候，不会对中断有响应。此时无法检测中断
    状态，或接收InterruptException异常。 
 < 