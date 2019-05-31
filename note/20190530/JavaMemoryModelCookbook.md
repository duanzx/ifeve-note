### Java内存模型Cookbook阅读笔记 [参考链接](http://ifeve.com/jmm-cookbook/)

核心思想：说明了在最简单的背景下 **指令重排**、**内存屏障**、**多处理器**等规则在JVM中存在的原因。

*   #### [指令重排](http://ifeve.com/jmm-cookbook-reorderings/)


> 1. **什么是指令重排？**     
   > CPU和编译器为了提升并行处理的效率,会按照一定的规则进行指令优化。如果当前的指令序列是(a,b,c),
   指令执行速度 a < c < b，则CPU可以按照 b -> c -> a 顺序执行。
   若CPU是三核处理器，则handler-1可执行b,handler-2执行c,handler-3执行a,
   这样等到b,c都执行完毕的时候,a也正好执行完。相比 a -> b -> c 顺序，提高了CPU的执行效率。
> 2. **指令重排的问题**        
   > 如果代码的逻辑之间存在一定的先后顺序，在并行处理的时候就会尝试二义性，不同的执行逻辑，得到不同的结果。        
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
> 3. **JMM中指令重排的规则**        
    
        
            
    


