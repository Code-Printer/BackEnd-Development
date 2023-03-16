# MQ  
MQ(message quenue)消息队列，遵循FIFO先进先出机制，用于上下游的通信。  
## MQ的特点  
1、流量削峰  
对高峰时的超出系统承载的请求做缓存，再输出  
2、子系统解耦  
使子系统之间不用在有强依赖性（还是利用缓存，将未处理的消息先放到队列中），提升系统的可用性  
3、异步处理（两个任务不会阻塞，同时执行）  
比如A需要调用B的服务，但B执行时间较长，A需要B的执行结果，以往有两个方法解决，一是A提供一个callBack方法，B执行完，调用该方法通知A，二是A去轮训。现在使用消息队列，在B执行完后，只需要发条消息给MQ，A订阅MQ就可以收到处理完成的结果信息。    
## RabbitMQ  
存储转发消息。由于erlang语言的高并发性，所以RabbitMQ的性能比较好，微秒级的时效性，可以处理数据量较少的场景   
### RabbitMQ的四大核心概念  
生产者、消费者、队列、交换机（指定接收的消息发往那个队列或丢弃）  
#### 名词介绍  
Broker：接收和分发消息的应用，RabbitMQ Server 就是 Message Broker。
Virtual host：出于多租户和安全因素设计的，把 AMQP 的基本组件划分到一个虚拟的分组中，类似于网络中的 namespace 概念。当多个不同的用户使用同一个 RabbitMQ server 提供的服务时，可以划分出多个 vhost，每个用户在自己的 vhost 创建 exchange／queue。
Connection：publisher／consumer 和 broker 之间的 TCP 连接。
Channel：如果每一次访问 RabbitMQ 都建立一个 Connection，在消息量大的时候建立TCPConnection 的开销将是巨大的，效率也较低。Channel 是在 connection 内部建立的逻辑连接，如果应用程序支持多线程，通常每个thread创建单独的channel进行通讯，AMQP method 包含了 channel id 帮助客户端和 message broker 识别 channel，所以 channel 之间是完全隔离的。Channel 作为轻量级的Connection 极大减少了操作系统建立 TCP connection 的开销。 
Exchange：message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key，分发消息到 queue 中去。常用的类型有：direct (point-to-point), topic (publish-subscribe) and fanout(multicast)。
Queue：消息最终被送到这里等待 consumer 取走。
Binding：exchange 和 queue 之间的虚拟连接，binding 中可以包含 routing key，Binding 信息被保存到 exchange 中的查询表中，用于 message 的分发依据。
