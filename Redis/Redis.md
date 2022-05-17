# Redis   
现实场景下数据量的爆发式增长和数据类型的复杂化，依靠传统的关系型数据库无法解决，非关系型数据库应运而生。
## 特点
(1) 支持数据的持久化，可以将数据保存在磁盘中，重启之后可以再次加载到内存中使用。    
(2) Redis中的值支持多种数据类型，支持String、list、set、zset、hash等数据结构。  

## Redis的五大数据结构及应用场景  
1、String：Redis中String类型的数据最大只能是512M   
应用场景：
计数器(可以使用incr命令实现自增操作，来计数，例如统计文章的阅读量和粉丝数)  
存储系统中的配置信息或Token令牌信息  
session共享(在分布式系统中，由于多台服务器，请求转发的每次分发到的服务器可能都不一样，就需要一个中间件来保存session的状态，可以把session信息保存在redis中)  
限流(需要对特定IP、用户限流，限制每分钟访问的次数)  
2、list：字符串列表，按照插入顺序排序    
应用场景：  
消息队列(使用lpush+brpop命令实现阻塞队列)  
分页查询(如果数据保存在缓存中可以使用list数据结构，联合命令：lrange key 0 19每页20条分页获取)  
3、Hash：是一个String类型的Filed以及value的映射表，类似于Java的Map(格式{hash1：{name:"zhangsan",age:20}，hash2:{name:"lisi",age:23}....)  
应用场景：  
存储用户信息、订单信息  
购物车功能(使用用户id作为缓存key，商品编码skucode作为hashkey，商品信息作为hashValue)  
4、set：无序集合且集合内元素唯一  
应用场景：  
统计公共好友、公共爱好  
统计访问用户的IP地址  
有返回随机数功能，故可以做抽奖功能  
5、zset：有score权值排序的set集合  
应用场景：   
用户得分/学生成绩排行榜功能

## Redis可以用作分布式锁的原因  
因为Redis是单线程，所以可以当做分布式锁(看一下分布式锁的原理)

## Redis可以用作消息队列的原因  
因为redis支持rpush和lpush、rpop和lpop操作，因此可以将请求按照队列的方式插入redis服务器中。
## 应用场景  
1、热点数据加速查询(主要场景)，如热点商品、热点信息等访问量较高的数据(主要使用LRU和LFU的缓存机制)  
2、即时信息查询，如公交到站信息、在线人数信息等  
3、时效性信息控制，如验证码控制、投票控制等  
4、分布式数据共享，如分布式集群架构中的session分离消息队列  

## 数据持久化存储  
1、概念：利用磁盘进行数据存储，在特定时间将数据进行恢复的工作。  
2、持久化的两种方式  
(1)快照方式(RDB(Redis DataBase))：将某个时间点的工作状态保存下来，恢复时可直接恢复指定时间点的工作状态。    
(2)日志方式(AOF(append only file))：将对数据的所有操作过程记录下来，恢复数据时重新执行这些操作。一个基于内存的key-value数据结构的NoSql数据库。  

## RDB与AOF的选择  
1、对比    
![result](https://static01.imgkr.com/temp/4b736ee3579649efb7584289dcc5cc35.png)  

2、AOF和RDB选择策略   
![result](https://static01.imgkr.com/temp/dcaabdb3f8434f04b6ffbad1105c03d2.png)  

## Redis基础知识  
1、Redis是单线程的  
2、Redis默认拥有16个数据库，数据库编号从0开始，默认使用0数据库  
3、select 数据库编号：可以切换使用的数据库  
4、dbsize查看当前数据库key的数量  
5、key * 查看数据库所有的key  
6、flushdb清空当前数据库  
7、flushall清空所有数据库  
8、Redis中所有数据库使用同一个密码，默认没有密码，Redis认为安全层面应该由Linux来保证  
9、Redis的默认端口6379   
## Redis数据库的存储空间  
![result](https://static01.imgkr.com/temp/1817f0eba0694fd69b22e6f9b097868c.png)  

## Redis的Linux操作及不同数据类型值的操作方法  
### 1、key(键)
![result](https://static01.imgkr.com/temp/638a22d2949b466e8b168be85d9e8e40.png)  
![result](https://static01.imgkr.com/temp/d7f90c0ce2f64577a2d656390b19c631.png)  

### 2、String  
一个key对应一个value；String可以包含任何数据，比如jpg图片等；String是Redis最基本的数据类型，一个String的value最大可支512M。   
![result](https://static01.imgkr.com/temp/d5b6fd08bb544b79aa66d45266509b39.png)  
![result](https://static01.imgkr.com/temp/7b5a5a94b90740838ff5b47d165f959a.png)  
### 3、List  
底层是一个字符串链表；可以从头或尾添加元素  
![result](https://static01.imgkr.com/temp/d7e3006a20d44a5ea852ea4d50f450b4.png)  
![result](https://static01.imgkr.com/temp/df7669a12499461bae72a17549512342.png)  
### 4、Set  
底层通过HashTable实现；是String类型的无重复值的无序集合  
![result](https://static01.imgkr.com/temp/661e5f1cd7114a2395f6f6905904b7e3.png)  
![result](https://static01.imgkr.com/temp/910818f2af884b72872a52e53963cf6c.png)  
### 5、Zset (有序集合)  
每个元素都会关联一个double类型的分数(score)；Redis通过分数自动的为集合中的 成员进行从小到大的排序；成员不可重复，分数可以重复  
![result](https://static01.imgkr.com/temp/5486c49180e54f20aa61442e03394de0.png)  
### 6、Hash表  
类似Java中的HashMap<String, Object>；是一个键值对集合；适合存储对象   
![result](https://static01.imgkr.com/temp/c826a01509d74fa3b03efd8df23ea0e3.png)   

### ubuntu上在redis客户端通过操作命令操作Redis的服务器  
1、命令行开启redis客户端  
```
~ redis-cli
```  
2、查看和删除redis服务器中的键  
```
> keys *   #查看redis服务器中的所有键  
> del keyName #删除该键的记录
```  
3、增加一条键值对和获取一个key值  
```
> set keyName Value  
> get keyName
```  
4、增加一条数字记录和让该数字自增  
```
> set keyName 2  
> INCR keyName  
```  
5、增加键的值为list类型  
```
> LPUSH keyName Value  #从该列表的左侧插入  
> RPUSH keyName Value #从该列表的右侧插入  
> LRANGE keyName 起始位置  终点位置  #从左至右打印该键的列表中的起止值
```  
6、增加一个键的值为hash表的记录和获取该记录数据  
```
> HSET keyName 键值对(keyName Value)  
> HGET keyName 键值对的键  #获取该键的值HashMap集合中对应键的值  
> HGETALL keyName  #获取该键的HashMap集合
```  
7、增加一条键的值为hash表的记录  
```
> HMSET keyName 键值对1 键值对2 ...  #增加一条键的值为hash表类型并一次性插入多个键值对
> HMGET keyName key1 key2 ... #获取keyName的hash表中多个键的值
```
## set指令与mset(多指令)指令的选择   
1、使用set存储n个值，需要操作n次指令，故所需时间：n个网络时间(发送给Redis服务器) + n个处理时间(处理操作) + n个网络时间(返回结果)  
2、使用mset存储n个值，需要操作1次指令，故所需时间：1个网络时间(发送) + n个处理时间(处理) + 1个网络时间(返回)  
结论：当操作的数据较少时，采用单指令；当处理的数据较多时，采用多指令。  

## 案例实践  
<font color="white">要求：</font>在微信聊天过程中，当接收到信息时，会将最新的消息显示在窗口顶端，当收到不同用户的消息时，会因为是否置顶以及消息到达的先后顺序产生不同的显示顺序。  
<font color="white">分析：</font>假设100这位用户将400与500两位好友消息置顶，当收到多个好友的消息时，它的微信聊天 窗口会产生怎样的显示顺序呢？  
<font color="white">解决思路:</font>  
(1) 将400与500两位用户放入set中(因为不重复)，表示置顶好友  
(2) 创建两个list列表(因为有顺序)，分别表示置顶用户的消息列表与非置顶用户的消息列表  
(3) 将上述两个list列表当作栈来使用  
(4) 当300用户给100发送消息时，检测到300用户并不属于set集合，将其压入普通list栈中  
(5) 当200用户和400用户给100发送消息时,检测到200不是置顶用户，将200压入普通栈中；检测到400是置顶用户，将400压入置顶栈中。    
(6) 当300用户再次给100发送消息时，判断得到300的消息需要压入普通list栈中，首先将栈中原来存在的300用户删除，此时200用户到达原来300用户的位置，然后将新的300用户的消息压入栈中。

![result](https://static01.imgkr.com/temp/b448b7319169424aa09ee04901a43c43.png)  

## Redis常用正则匹配的键查询指令  
![result](https://static01.imgkr.com/temp/2823052af78247ea925ba0bec2db0831.png)  

## Jedis的使用  
1、首先改配置，需要在redis.conf配置文件中修改配置，需要重启Redis服务方可生效  
![result](https://static01.imgkr.com/temp/615dd9e1a3a4492f8ed2518127150a84.png)  

2、改主机地址，必须指定绑定的主机地址方可使用Redis  
![result](https://static01.imgkr.com/temp/5ba26dbfe9844a8f847116fe595e2da1.png)  
3、改端口，开放6379端口  
![result](https://static01.imgkr.com/temp/dcde9136234b4b178dcdfe440624f604.png)  
4、创建Java项目并导入jar包  
![result](https://static01.imgkr.com/temp/1d6538bc76e948a79decde783ac30b0c.png)  

5、src目录下创建redis.properties配置文件  
```properties
#最大连接数
redis.maxTotal=50
#默认开启的活跃连接数
redis.maxIdel=10
#Linux的ip地址
redis.host=192.168.200.130
#redis的端口号
redis.port=6379
```  
6、创建Jedis的工具类JedisUtils  
```java
public class JedisUtils {

    // 将从配置文件读取的配置信息赋予如下变量
    private static int maxTotal;
    private static int maxIdel;
    private static String host;
    private static int port; // 端口号为int类型

    // Jedis的连接池配置
    private static JedisPoolConfig jedisPoolConfig;

    // Jedis连接池
    private static JedisPool jedisPool;

    static {
        // 读取redis.properties配置文件
        ResourceBundle bundle = ResourceBundle.getBundle("redis");
        maxTotal = Integer.parseInt(bundle.getString("redis.maxTotal"));
        maxIdel = Integer.parseInt(bundle.getString("redis.maxIdel"));
        host = bundle.getString("redis.host");
        port = Integer.parseInt(bundle.getString("redis.port"));

        // Jedis连接池配置
        jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxTotal(maxTotal);
        jedisPoolConfig.setMaxIdle(maxIdel);
        jedisPool = new JedisPool(jedisPoolConfig, host, port);
    }

    public static Jedis getJedis() {
        return jedisPool.getResource();
    }
}
```  
7、创建测试类JedisTest  
```java
public class JedisTest {
    public static void main(String[] args) {
        // 1. 获取Jedis对象
        Jedis jedis = JedisUtils.getJedis();

        // 2. 执行操作，Jedis中操作的方法名与Linux中命令行工具中的指令同名,添加set集合类型的数据
        jedis.sadd("key1", "abc", "abc", "def");
        Long key1 = jedis.scard("key1");//获取key1元素在集合中的个数
        System.out.println("运行结果：" + key1);

        // 3.关闭连接
        jedis.close();
    }
}
```  
8、Linux中进行测试  
![result](https://static01.imgkr.com/temp/233501bf88ef4b1bbc332aaefbb55c7f.png)  

## Redis可视化工具  
1、安装软件  
![result](https://static01.imgkr.com/temp/1d9a511ef2c84f5b96a0706c1ddeaded.png)  

2、运行软件  
![result](https://static01.imgkr.com/temp/c16255bc37e5422cbb600c8bf374e8f4.png)   

## RDB  
1、对redis.conf配置文件进行修改 (修改配置文件后需要重启Redis)  
(1) 修改内存中数据保存的文件的名称，默认值为dump.rdb  
![result](https://static01.imgkr.com/temp/a35666544369432fbcd8109dd4be4bcb.png)  
(2) 修改rdb文件保存的目录   
![result](https://static01.imgkr.com/temp/65f068c6a1af42bb9844f360f2a15768.png)  

2、执行save指令即可将内存中的数据保存到/opt/redis-3.0.4/目录的dump.rdb文件中。  
3、再次启动redis服务即可自动读取rdb文件中的数据并加载到内存。  
4、save指令工作原理  
Redis是单线程的，故执行save指令会阻塞其之后的命令的执行(可能多人操作同一个Redis 服务器)，如果要保存的数据较多时，会导致之后的命令长时间阻塞，故一般不使用save指令。  
5、bgsave指令可以让保存操作在后台执行，让redis服务可以继续执行其之后的指令，使用较多。  
6、bgsave指令工作原理  
![result](https://static01.imgkr.com/temp/3c1d20ddf8cd40b4b168570d93121efd.png)  

7、配置自动保存 (修改配置文件后需要重启Redis)  
![result](https://static01.imgkr.com/temp/a8fc0fbaa0fc4ccf8077b68be5b49585.png)   

8、自动保存方式的注意点  
(1) get操作没有导致key发生变化  
(2) 对存在的key修改才算发生变化  
(3) set k1 v1，set k1 v1认为key的值发生变化  
(4) 配置方式执行的是bgsave指令  
9、RDB两种指令的对比  
![result](https://static01.imgkr.com/temp/baea0553db9e4a078d2bf484eaa49695.png)    
10、RDB缺点  
(1) 基于快照思想，每次读写都是全部数据，当数据量较大时，效率非常低  
(2) 基于fork创建子进程，内存产生额外的消耗  
(3) 宕机带来数据丢失风险(可能某个时间点的数据未保存)  
## AOF  
1、AOF:以日志的方式记录每次改变数据的操作命令，重启之后执行AOF中保存的命令恢复数据，较为主流。

2、对redis.conf配置文件进行修改 (修改配置文件后需要重启Redis)  
![result](https://static01.imgkr.com/temp/0f597fe00892455eaeebc3a38d16b96e.png)   

3、AOF执行策略  
(1) everysec (每秒)
每秒将缓冲区的指令写入aof文件中，宕机会丢失0-1秒的数据，性能高，建议使用  
(2) always (每次)
每次执行指令都将其写入aof文件中，数据零失误，性能较低，不建议使用    
(3) no (系统控制)
由操作系统控制写入aof文件的时间，不建议使用    
注意：i. 只有使得key变化的指令才记录  
ii. 重启之后自动从aof文件中读取指令并执行  
iii. select指令虽然没有对key进行修改，但仍需记录，以知道数据存储的位置  

##  AOF重写  
1、概念：AOF文件中已经记录的对同一数据的若干条操作的记录转换为数据最终结果对应指令的记录  
![result](https://static01.imgkr.com/temp/67e0ce6bf9c742bbb73084c926d30813.png)    

2、AOF重写作用  
1、降低磁盘占用量，提高磁盘利用率  
2、提高持久化效率，降低持久化写的时间，提高读写性能  
3、降低数据恢复用时，提高数据回复效率  
3、为防止数据量过大导致缓冲区溢出，合并后的每条指令最多写入64个元素  
4、AOF重写方式  
(1) 手动重写，执行bgrewriteaof指令  
![result](https://static01.imgkr.com/temp/30b3c517ea4e47418d191cf32eb49d28.png)  

(2) 自动重写，修改配置文件  
![result](https://static01.imgkr.com/temp/54c8d863fdd44b90beeb76e0b4b8ebad.png)  
