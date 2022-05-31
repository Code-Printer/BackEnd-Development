# 主从复制(集群的基础)  
## 主从复制概念  
为了避免单点Redis服务器故障，准备多台服务器，进行分布式主从部署。将主服务器master数据<font color='white'>复制</font>多个副本保存在副服务器slave上，连接在一起，并保证数据是<font color='white'>同步</font>的；好处是即使有其中一台服务器宕机，其他服务器依然可以继续提供服务，实现Redis的高可用，同时实现数据备份。  
### 多台服务器master与slave的对应关系  
![result](https://static01.imgkr.com/temp/971d021c0372422085b2951070457d35.png)  

1、主从复制即只能将主机master服务器上的数据单向备份到从机slave上；  
2、一个master可以有多个slave，一个slave只能有一个master；  
3、master负责**写数据**，然后将改变的数据自动同步到slave上；slave负责**读数据**。  
### 主从复制的好处  
1、负载均衡：提高服务器的读写负载能力：使用读写分离策略，master负责写，slave负责读   
2、数据备份：slave会备份master的数据  
3、高可用(集群)：基于主从复制，构建分布式哨兵和集群模式，实现Redis高可用。  
## Redis主从复制步骤  
![result](https://static01.imgkr.com/temp/0dca3657cbcc4ba8a41d0e639784069b.png)   

## 主从复制流程   
1、建立连接：从机slave主动连接主机master，发送同步指令到主机   
2、数据同步：主机master接收到指令，使用RDB方式将数据全量复制同步到连接自己的slave服务器上   
3、增量复制：主机master通过aof将改变数据的命令依次同步到slave从机上   
![result](https://static01.imgkr.com/temp/7d1555593ab24a7880a5a988d19994ff.png)  

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
### 命令传播  
当master数据库状态被修改后，导致主从服务器数据库状态不一致，此时需要让主从数据同步到一致的状态，同步的动作称为命令传播。  
1、增量复制的三个元素：  
(1)、服务器的运行id (run id)：服务器运行ID是每一台服务器每次运行的身份识别码，一台服务器多次运行可以生成多个运行id。作用是运行id用于在服务器间进行传输时识别身份。 
(2)、主服务器的复制积压缓冲区：是一个先进先出的队列，用于存储服务器执行过的变更数据的命令，每次传播命令，master都会将待传播的命令记录下来，并存储在复制缓冲区  
(3)、主从服务器的复制偏移量：一个数字，描述复制缓冲区中的指令字节位置，分为master复制偏移量(发给所有slave的指令字节位置)和slave复制偏移量(记录slave接收master发送过来的指令字节对应的位置)    
## 心跳机制   
命令传播阶段后，master与slave间需要进行信息交换，使用心跳机制进行维护。   
### master心跳  
1、指令：PING  
2、周期：由配置文件中repl-ping-slave-period参数决定，默认10秒   
3、作用：判断slave是否在线  
4、查询：master使用INFO replication 指令可以列出从服务器列表，获取slave最后一次向master发送命令距离现在过了多少秒，lag 项为 0 或 1 视为正常   
### slave心跳  
1、指令：向master发送指令REPLCONF ACK {offset}（offset是从服务器进行复制的当前的复制偏移量）  
2、周期：1秒  
3、作用：(1):向master汇报自己的复制偏移量，获取最新的数据变更指令；(2):判断master是否在线  
### 注意事项  
1、当slave多数掉线或者延迟过高时，master为了保证数据稳定性，将会拒绝所有信息的同步操作。  
2、slave的数量和延迟信息通过向mater发送 REPLCONF ACK 指令告知master    
3、通过两个参数设置(min-slaves-to-write 2;min-slaves-max-lag 8(表示当slave个数少于2个，或者所有slave的延迟都大于等于8秒时，强制关闭master写功能，停止数据同步))  
## 主从复制实战环境配置    
```
>info replication  //查看当前redis服务器信息
```
搭建两个从机slaver和一个主机master：
1、复制三个redis.config文件，修改每个配置文件的内容(重点修改端口号、pid、log文件名、dump.rdb名)  
2、使用redis-server 配置文件的命令，分别在三个窗口启动三个redis服务器，并使用redis客户端连接服务器，此时三台都是主机master，需要配置其中的两台为从机slave    
3、在其中的两台客户端上使用如下命令指定主机地址和端口号，当前服务器自动变为从机slave      
```
slaveof host port
```
4、真实的开发场景中是在配置文件中配置从机的配置文件  
![](https://mrggz.oss-cn-hangzhou.aliyuncs.com/img/202205311047625.png?x-oss-process=style/null)  
