##  SpringBoot项目中整合Redis  
1. 首先在IDEA中新建SpringBoot项目  
![](https://mrggz.oss-cn-hangzhou.aliyuncs.com/img/202205231753440.png?x-oss-process=style/null)  
2. 选择对应的项目依赖，点击finish，创建项目   
3. 在resource目录下的application.properties文件中配置redis的连接信息  
```properties
spring.redis.host=127.0.0.1
spring.redis.port=6379
```  
4. 测试  
```java
@Autowired
    RedisTemplate redisTemplate;
    @Test
    void contextLoads() {

        //opsForValue 操作字符串
        //opsForList 操作列表list
        //opsForSet 操作集合Set
        //opsForZSet 操作集合ZSet
        //opsForHash 操作hash
        //...
        redisTemplate.opsForValue().set("mykey","hello");  //opsForValue对字符串类型操作
        System.out.println(redisTemplate.opsForValue().get("mykey"));
        //获取redis连接对象，进行数据库的清除操作
        RedisConnection redisConnection =  redisTemplate.getConnectionFactory().getConnection();
        redisConnection.flushAll();
        redisConnection.flushDb();
    }
```  
5. 自定义对象序列化的RedisTemplate对象  
```java

```
6. 封装自己的RedisUtils  
```java

```
### SpringBoot2.x之后将原本的Jedis替换成了lettuce  
jedis采用直连，需要使用jedis连接池，避免多线程操作的不安全性，更像BIO的方式  
lettuce采用netty(提供NIO的客户端与服务器端的通信方式)，实现多线程的数据共享，不存在线程安全问题。   
redis的自动配置源码分析：  
1. 首先在项目的external Libraries中找到springbootautoconfigure包然后找到META-INF包下的spring.factory文件，找到并进入RedisAutoConfiguration源码  
```java
@Bean
    @ConditionalOnMissingBean(
        name = {"redisTemplate"}
    )
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        //没有设置的初始redistemplate，redis的对象存储都需要进行序列化  
        //key和value数据类型泛型都是object，自己使用需要进行强制转换
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    @ConditionalOnMissingBean
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
```  

