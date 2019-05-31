### Java内存模型Cookbook阅读笔记 参考链接：[并发编程网-JMM Cookbook](http://ifeve.com/jmm-cookbook/)

核心思想：说明了在最简单的背景下 **指令重排**、**内存屏障**、**多处理器**等规则在JVM中存在的原因。

*   #### [指令重排](http://ifeve.com/jmm-cookbook-reorderings/)


> 1. **什么是指令重排？**     
   > CPU和编译器为了提升并行处理的效率,会按照一定的规则进行指令优化。如果当前的指令序列是(a,b,c),
   指令执行速度 a < c < b，则CPU可以按照 b -> c -> a 顺序执行。
   若CPU是三核处理器，则handler-1可执行b,handler-2执行c,handler-3执行a,
   这样等到b,c都执行完毕的时候,a也正好执行完。相比 a -> b -> c 顺序，提高了CPU的执行效率。
> 2. **指令重排的问题**        
   > 如果代码的逻辑之间存在一定的先后顺序，在并行处理的时候就会产生二义性，不同的执行逻辑，得到不同的结果。
   在单线程执行时，这些指令不允许重排： 
         <table>
            <tr>
                <th>名称</th><th>代码</th>
            </tr>
            <tr>
                <td>先写后读</td>
                <td>a=1;b=a</td>
            </tr>
            <tr>
                <td>先写后写</td>
                <td>a=1;a=2</td>
            </tr>
            <tr>
                <td>先读后写</td>
                <td>a=b;b=1</td>
            </tr>
        </table>            
> 3. **JMM中禁止指令重排的规则**        
   > 主要有两种：对字段(包括数组中的元素)的存取指令(load,store)；对锁(lock)的控制指令。具体指令有：        
        <ul>
            <li>
                Normal Load：对非Volatile字段的读取，getField,getStatic,array load;   
            </li>
            <li>
                Normal Store：对非Volatile字段的存储，putField,putStatic,array store;   
            </li>
            <li>
                Volatile Load：对Volatile字段的读取，getField,getStatic;   
            </li>
            <li>
                Volatile Store：对Volatile字段的存储，putField,putStatic;   
            </li>
            <li>
                MonitorEnters：包括进入同步块Synchronized的方法
            </li>
            <li>
                MonitorExits：包括退出同步块Synchronized的方法
            </li>
        </ul>
        <table>
            <tr>
                <th>能否重排</th>
                <th colspan="3">第二个操作</th>
            </tr>
            <tr>
                <th>第一个操作</th>
                <th>Normal Load,Normal Store</th>
                <th>Volatile Load,MonitorEnter</th>
                <th>Volatile Store,MonitorExit</th>
            </tr>
            <tr>
                <td>Normal Load,Normal Store</td>
                <td></td>
                <td></td>
                <td>NO</td>
            </tr>
            <tr>
                <td>Volatile Load,MonitorEnter</td>
                <td>NO</td>
                <td>NO</td>
                <td>NO</td>
            </tr>
            <tr>
                <td>Volatile Store,MonitorExit</td>
                <td></td>
                <td>NO</td>
                <td>NO</td>
            </tr>
        </table>
    提示：上述关于Volitale,Monitor的规则仅仅适用于可能被多线程访问的变量和对象。如果一个编译器可以证明，一个
    锁只被单线程访问，那么这个锁就可以去掉。如果一个Volitale变量只被单线程访问，这个变量也可以被当作是普通变量。      
    不能被同一个内存地址上的load指令和之后紧跟着的store指令重排，但是可以被两个不同的内存地址上的load和store
    指令进行重排。
> 4. **Final字段的重排规则**    
            
    


