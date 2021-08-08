# Redis的哨兵模式  
## 哨兵模式(主要就是三个阶段：监控、通知、故障转移)  
### 哨兵的概念  
哨兵是一个分布式系统的监控服务器，用于对主从结构中的redis服务器进行监控，当master出现宕机时通过选票机制选取新的master并将所有的slave连接到新的master。  
![result](https://static01.imgkr.com/temp/383c2fe864f94026bbdb6348f1994010.png)  
### 哨兵的作用   
1、监控：监控master是否存活(宕机)，监控slave的运行情况(服务器地址、端口号、偏移量、运行id...)。  
2、通知：当被监控的某一服务器出现问题时，及时通知其他哨兵和服务器  
3、故障转移：当master出现宕机时，通过投票选举从slave中选出新的master，将其他的slave连接到新的master，并告知客户端新的master地址。(一般哨兵的数量为单数，方便投票)  
### 服务器启用哨兵  
1、修改哨兵的配置文件 sentinel.conf ，配置文件位于Redis目录下，如下图所示：  
![result](https://static01.imgkr.com/temp/d68e568a316443ba9f8dfc9975345980.png)    
![result](https://static01.imgkr.com/temp/df82ee33fcb444768dbb60f9b155fb81.png)  
部分配置的含义如下图所示：  
![result](https://static01.imgkr.com/temp/34fc6d4d406b4fee9fdd6a05fad8ba71.png)  
2、启动哨兵(redis-sentinel filename)     
filename指的是配置文件名，每个哨兵都要配置自己的配置文件, 配置并启动哨兵之后，主服务器宕机之后，会自动的执行投票、主从切换等过程。  
### 哨兵原理  
哨兵在进行主从切换过程中主要经过三个阶段：监控-通知-故障转移  
1、监控阶段：信息同步  
2、通知阶段：确保联通，无宕机  
3、故障转移：  
(1):哨兵发现master宕机，依次标记主观和客观下线  
(2):哨兵竞选选举者
(3):选举人哨兵决定新的master
(4):将slave与新的master连接，原master恢复之后作为slave连接到新master。  
#### 监控  
同步各节点的状态信息，这些节点包括master(获取master的状态信息，主要是获取slave和已有的哨兵信息)、slave(运行id、角色slave、master主机号和端口号、偏移量)、哨兵(新添加的哨兵与已存在哨兵进行信息同步)。  
具体的工作流程原理如下图：  
![result](https://static01.imgkr.com/temp/aaafb7edc10e4a599d85babb00a0ca17.png)  
1、哨兵1向master发送info指令之后，会建立一个cmd连接，创建的连接是用来发送命令的。  
2、创建好cmd连接之后，会在哨兵1这一端保存目前他所获得的所有信息，另一端master也会保存自己持有的信息。  
3、然后哨兵1根据从master获取的关于salve的信息，向slave发送info指令，得到salve的信息，丰富所保存的信息。    
4、当新增一个新的哨兵2时，哨兵2向master发送info指令，建立cmd连接，根据master中的信息可以得到之前已经存在的哨兵1，在master一端保存已经获得的信息。然后判断哨兵1是否在线，与哨兵1建立连接，二者互相交换各自的信息，并且双方会持续的ping，保证他们之间是畅通的。  
5、哨兵2根据从master获得的slave信息，再从slave获取信息，丰富自己所保存的信息，再新增一个哨兵3时，与之前的过程类似，最终三个哨兵建立起了关系网。  
6、关系网中三者会互相交换、发送信息，关系网中的这种工作模式称为发布订阅模式。  
#### 通知  
某个哨兵通过建立的cmd连接，向master、slave发送 hello 指令，得到他们的信息(是否在线)，然后在关系网中共享：    
![result](https://static01.imgkr.com/temp/02d9900f7833475097804fbc5fe0e78a.png)  
#### 故障转移(先是决定由哪个哨兵来选，再是决定由哪个slave来当master)  
当故障发生时，断开宕机的master与slave的连接，选取一个slave作为新的master，将其他slave连接新的master，并告知客户端新的master服务器地址，当宕机的原master恢复后，作为slave连接到新的master上。  
具体流程如下图：  
![result](https://static01.imgkr.com/temp/334351119aad4d268f5a2e7c17b34bdf.png)  
1、当哨兵1发送hello指令始终得不到master的回应，就会主观判断master宕机，标记sdown（仅有一个哨兵认为master宕机）  
2、哨兵1在关系网中传递消息，向其他哨兵同步自己的信息(关于master服务器宕机)  
3、其他哨兵得到消息之后，全部向master发送消息，确定master是否宕机，他们也会将自己得到的结果发送给关系网中的其他哨兵，确定master宕机后，将标记改为odown（所有哨兵探测之后，超过一半的哨兵认为master宕机）  
4、当认定master宕机之后，此时需要选举新的master，需要通过哨兵之间的选举决定哪个哨兵可以当选为选举人：  
(1): 每个哨兵手里都有一票，每个哨兵都会发出一个指令，在内网里边告诉其他哨兵自己当选举人，比如说sentinel1和sentinel2发出这个选举指令，那么sentinel3接收到他们的申请之后，sentinel3就会把自己的一票投给其中一方，根据到达的先后顺序投票，最终会有一个选举结果，获得票数最多的哨兵会被获选为选举者。  
(2):在选举过程中有可能会存在失败的现象，比如第一轮没有选举成功，那就会接着进行第二轮第三轮直到完成选举  
5、当完成哨兵之间的选举之后，接下来就要由选举胜出的哨兵去slave中挑选一个，将其变成新的master，具体的流程如下所述：  
(1):哨兵在服务器列表中挑选备选master的原则：先排除，后选择(排除不在线、响应慢、与原master断开事件久的、优先级低的、偏移量小的) ，若还没有选出，则由uuid选出。   
(2):选出新的master之后，哨兵发送指令给新的master服务器(1、向新的master发送 slaveof no one 指令 2、向其他slave发送slaveof 新masterIP端口 指令)。  
(3):告诉其他的哨兵新master是谁。  


