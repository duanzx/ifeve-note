#### 思考问题
1. RocketMQ由几部分组成以及每个组件的作用
2. RocketMQ消息如何保证可靠性以及高可用性
3. RocketMQ消息种类以及怎么保证消息有序
#### RocketMQ与kafka对比
#### 数据可靠性
> RocketMQ支持异步实时刷盘，同步刷盘，同步复制，异步复制   
kafka使用异步刷盘，异步复制/同步复制   

#### ActiveMQ、Kafka、RocketMQ比较

<table>
    <tr>
        <th>功能</th>
        <th>ActiveMQ</th>
        <th>Kafka</th>
        <th>RocketMQ</th>
    </tr>
    <tr>
       <td>客户端SDK</td> 
       <td>Java，.NET，C++，etc</td>
       <td>Java，Scala，etc.</td>
       <td>Java，C++，Go</td> 
    </tr>
    <tr>
       <td>协议和规范</td> 
       <td>消息Push模型，支持OpenWire，STOMP，AMQP，MQTT，JMS</td>
       <td>Pull模型，支持TCP</td>
       <td>Pull模型，支持TCP，JMS，OpenMessaging</td>
    </tr>
    <tr>
       <td>消息排序</td> 
       <td>单个消费者或单个队列内的消息可以排序</td>
       <td>同一个分区内的消息可以排序</td>
       <td>可确保消息严格有序，并且可以优雅扩展</td>   
    </tr>
    <tr>
       <td>消息预定</td> 
       <td>支持</td>
       <td style="color:red">不支持</td>
       <td>支持</td>
    </tr>
    <tr>
       <td>消息批处理</td> 
       <td style="color:red">不支持</td>
       <td>支持，使用异步生产者</td>
       <td>支持，使用同步模式以避免消息丢失</td> 
    </tr>
    <tr>
       <td>消息广播</td> 
       <td>支持</td>
       <td style="color:red">不支持</td>
       <td>支持</td>
    </tr>
    <tr>
       <td>消息过滤</td> 
       <td>支持</td>
       <td>支持，可以使用Kafka Streams来过滤消息</td>
       <td>支持，基于SQL92的属性过滤表达式</td> 
    </tr>
    <tr>
       <td>消息重发</td> 
       <td style="color:red">不支持</td>
       <td style="color:red">不支持</td>
       <td>支持</td>
    </tr>
    <tr>
       <td>消息存储</td> 
       <td>使用JDBC和高性能的日志(如levelDB,kahaDB)，支持非常快速的持久化</td>
       <td>高性能文件存储，零复制</td>
       <td>高性能和低延迟的文件存储</td>   
    </tr>
    <tr>
       <td>消息追溯</td> 
       <td>支持</td>
       <td>支持，偏移量表示</td>
       <td>支持，可以用时间戳、偏移量表示</td>
    </tr>
    <tr>
       <td>消息优先级</td> 
       <td>支持</td>
       <td style="color:red">不支持</td>
       <td style="color:red">不支持</td>
    </tr>
    <tr>
       <td>高可用和故障转移</td> 
       <td>支持，要依赖于存储，如果使用kahaDB,还要用到Zookeeper服务器</td>
       <td>支持，需要一个Zookeeper服务器</td>
       <td>支持，使用主从模型</td>
    </tr>
    <tr>
       <td>消息跟踪</td> 
       <td style="color:red">不支持</td>
       <td style="color:red">不支持</td>
       <td>支持</td> 
    </tr>
    <tr>
       <td>配置</td> 
       <td>默认配置为低级别。使用时，需要优化配置参数</td>
       <td>Kafka使用key-value进行配置</td> 
       <td>开箱即用，用户只需要注意一些额外配置</td>
    </tr>
    <tr>
       <td>管理和操作工具</td> 
       <td>支持</td>
       <td>支持，使用terminal命令</td>
       <td>Web管理页面和terminal命令</td>   
    </tr>
</table>
