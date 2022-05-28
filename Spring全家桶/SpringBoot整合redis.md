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
5. 定义支持序列化的对象和自定义通用的RedisTemplate对象，定义其序列化对象的方式    
```java  
//实现序列化接口，支持对象可被序列化
public class User implements Serializable{
    public String name;
    public int age;

}
```
```java
@Configuration
public class RedisConfig {

    //编写通用的redisTemplate模板
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        //为了开发方便，一般redis使用的键值对数据类型为<String,Object>
        RedisTemplate<String, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);

        //json的序列化配置
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        //String的序列化配置
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();

        template.setKeySerializer(stringRedisSerializer);  //设置key的序列化采用String方式
        template.setHashKeySerializer(stringRedisSerializer); //设置Hashkey的序列化采用String方式
        template.setValueSerializer(jackson2JsonRedisSerializer); //设置redis的value序列化采用jackson的方式
        template.setHashValueSerializer(jackson2JsonRedisSerializer); //redistemplate的Hashvalue默认使用jdk的序列化方式，这里改用jackson的方式
        template.afterPropertiesSet();
        return template;
    }
}
```
6. 封装自己的RedisUtils  
```java
@Component
public class RedisUtils {
    @Autowired
    private RedisTemplate redisTemplate;

    /**
     * 设置key的缓存过期时间
     * @param key
     * @param time
     * @return
     */
    public boolean expire(String key,long time){
        try{
            if (time > 0){
                redisTemplate.expire(key,time, TimeUnit.SECONDS);
            }
            return true;
        }catch (Exception e){
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据key获取key的缓存过期时间
     */

    public long getExpire(String key){
        return redisTemplate.getExpire(key,TimeUnit.SECONDS);
    }

    /**
     * 判断key是否存在
     */

    public boolean hasKey(String key){
        try{
            return redisTemplate.hasKey(key);
        }catch (Exception e){
            e.printStackTrace();
            return false;
        }
    }

    /**
     *根据key删除缓存
     */
    public void del(String... key){
        if (key != null && key.length > 0){
            if (key.length == 1){
                redisTemplate.delete(key);
            }else{
                redisTemplate.delete(CollectionUtils.arrayToList(key));
            }
        }
    }

    /**
     * 根据key获取缓存值
     */
    public Object get(String key){
        return key == null ? null : redisTemplate.opsForValue().get(key);
    }

    /**
     * 存入缓存
     */

    public boolean set(String key,Object value){
        try{
            redisTemplate.opsForValue().set(key,value);
            return true;
        }catch (Exception e){
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 缓存放入并设置有效时间
     */
    public boolean set(String key, Object value,long time){
        try{
            if (time > 0){
                redisTemplate.opsForValue().set(key,value,time);
            }else{
                set(key,value);
            }
            return true;
        }catch (Exception e){
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 值递增
     */
    public long incr(String key,long delta){
        if (delta < 0){
            throw new RuntimeException("递增因子须大于0");
        }
        return redisTemplate.opsForValue().increment(key,delta);
    }

    /**
     * 值递减
     * delta 减的值
     */

    public long decr(String key,long delta){
        if (delta < 0){
            throw new RuntimeException("递减因子须大于0");
        }
        return redisTemplate.opsForValue().increment(key,-delta);
    }

    /**
     * 获取Hash数据类型的item的值
     */
    public Object hget(String key,String item){
        return redisTemplate.opsForHash().get(key,item);
    }

    /**
     * 获取key对应的hash集合
     */
    public Map<String,Object> hmget(String key){
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * 根据键设置一个hash集合类型的value
     */

    public boolean hmset(String key,Map<String,Object> map){
        try{
            redisTemplate.opsForHash().putAll(key,map);
            return true;
        }catch (Exception e){
            e.printStackTrace();
            return false;
        }
    }

    /**
     * HashSet设置缓存时间
     */
    public boolean hmset(String key,Map<String,Object> map,long time){
        try{
            redisTemplate.opsForHash().putAll(key,map);
            if (time > 0){
                expire(key,time);
            }
            return true;
        }catch (Exception e){
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 在对应key的hash集合中存放hash对
     */
    public boolean hset(String key,String filed,Object value){
        try{
            redisTemplate.opsForHash().put(key,filed,value);
            return true;
        }catch (Exception e){
            e.printStackTrace();
            return false;
        }
    }

}
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

