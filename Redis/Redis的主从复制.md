# 主从复制  
## Redis主从复制步骤  
![result](https://static01.imgkr.com/temp/0dca3657cbcc4ba8a41d0e639784069b.png)  

## 主从复制概念  
为了避免单点Redis服务器故障，准备多台服务器，进行分布式主从部署。将主服务器master数据<font color='white'>复制</font>多个副本保存在副服务器slave上，连接在一起，并保证数据是<font color='white'>同步</font>的；好处是即使有其中一台服务器宕机，其他服务器依然可以继续提供服务，实现Redis的高可用，同时实现数据备份。  
### 多台服务器master与slave的对应关系  
![result](https://static01.imgkr.com/temp/971d021c0372422085b2951070457d35.png)  

1、主从复制即将master服务器上的数据备份到slave上  
2、一个master可以有多个slave，一个slave只能有一个master  
3、master主要负责写数据，然后将改变的数据自动同步到slave上；slave主要负责读数据  
### 主从复制的好处  
1、提高服务器的读写负载能力：使用读写分离策略，master负责写，slave负责读  
2、负载均衡：slave分担master负载，并可以根据需求改变slave的数量，提高系统的并发量(同时可读操作数)和数据吞吐量  
3、快速故障恢复：当master出现故障时，slave可以转变成master提供服务(哨兵机制)。  
4、数据备份：slave会备份master的数据  
5、高可用：基于主从复制，构建分布式哨兵和集群模式，实现Redis高可用。  
## 主从复制流程  
1、建立连接：slave主动连接master  
2、数据同步：master将数据同步到第一次连接自己的slave服务器上  
3、命令传播：master的数据变化同步到slave上  
![result](https://static01.imgkr.com/temp/7d1555593ab24a7880a5a988d19994ff.png)  

主从复制的四种指令：  
![result](https://static01.imgkr.com/temp/1285d3059c3f4bac91a5dfb7d1618211.png)  

### 建立连接  
1、建立slave到master的连接，使master能够识别slave，并保存slave端口号  
2、流程图如下：  
![result](https://static01.imgkr.com/temp/e11d314bad2948f1873845176aef8a19.png)    
此时状态：slave保存master的地址与端口，master保存slave的端口，二者可以通信。  
3、slave连接master，有三种方式：  
(1)slave向master发送命令：slaveof masterip masterport   
(2)启动slave服务器时进行连接：redis-server redis.conf --slaveof masterip masterport  
(3)启动slave服务器时自动连接(主流)，slave的配置文件中添加：slaveof masterip masterport  
注：slave、master的客户端分别执行 info 指令，可以查看连接信息。  
4、主从断开连接：断开slave与master的连接，slave断开连接后，不会删除已有数据，只是不再接受master发送的数据。(slave客户端执行：slaveof no one)   
### 数据同步  
1、整个过程分为两大块：全量复制和增量复制（部分复制）。全量复制是当slave第一次连接master时的数据同步操作(发送RDB文件方式)；增量复制是slave已经连接了master之后master的数据发生部分改变后，进行数据同步(发送AOF文件的方式)。  
2、流程图如下：  
![result](https://static01.imgkr.com/temp/aeec13b1939949818fdba589f3a384a8.png)    
此时slave拥有master端全部数据，master保存slave当前数据同步的位置,完成了从master到slave的数据同步  
3、数据同步阶段master的说明：  
(1)、如果master数据量巨大，数据同步阶段应避开流量高峰期，避免造成master阻塞，影响业务正常执行。  
(2)、复制缓冲区大小设定不合理会导致数据溢出(假如进行全量复制周期太长，缓冲区溢出，进行部分复制时发现缓冲区数据丢失，必须进行第二次全量复制，致使slave陷入死循环状态。), master配置文件中设置缓存区大小：repl-backlog-size ?mb。  
(3)、master单机内存占用主机内存的比例不应过大，建议使用50%-70%的内存，留下30%-50%的内存用于执行bgsave命令和创建复制缓冲区。  
4、数据同步阶段slave的说明  
(1)、为避免slave进行全量复制、部分复制时服务器响应阻塞或数据不同步，建议关闭此期间的对外服务(slave配置文件中进行对外服务的开关：slave-serve-stale-data yes|no)  
(2)、多个slave同时对master请求数据同步，master发送的RDB文件增多，会对带宽造成巨大冲击，如果master带宽不足，数据同步需要根据业务需求，适量错峰  
(3)、slave过多时，建议调整拓扑结构，由一主多从结构变为树状结构（上一层给下一层的服务器传递数据），注意使用树状结构时，由于层级深度，导致底层的slave与最顶层master间数据同步延迟较大，数据一致性变差，应谨慎选择。  
  