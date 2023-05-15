# SpringBoot高级  
## SpringBoot的Cache  
Java Caching定义了5个核心接口，分别是CachingProvider, CacheManager, Cache, Entry
和 Expiry。
• CachingProvider定义了创建、配置、获取、管理和控制多个CacheManager。一个应用可
以在运行期访问多个CachingProvider。
• CacheManager定义了创建、配置、获取、管理和控制多个唯一命名的Cache，这些Cache
存在于CacheManager的上下文中。一个CacheManager仅被一个CachingProvider所拥有。
• Cache是一个类似Map的数据结构并临时存储以Key为索引的值。一个Cache仅被一个
CacheManager所拥有。
• Entry是一个存储在Cache中的key-value对。
• Expiry 每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期
的状态。一旦过期，条目将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。  
## 关于Cache的注解  
Cache 缓存接口，定义缓存操作。实现有：RedisCache、EhCacheCache、ConcurrentMapCache等
@Cacheable 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存（当缓存中没有该缓存需要的值，会执行方法，结果放在缓存中，当缓存中有该key的结果，就不会执行该方法）
@CacheEvict 根据key清空缓存
@CachePut 保证方法被调用，又希望结果被缓存。（不管缓存中是否有key的结果都会执行方法，在注解的方法执行后，执行cachePut方法）
@EnableCaching 开启基于注解的缓存