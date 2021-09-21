#Redis的项目业务应用
## Redis业务中常用的几个方法   
1、键的操作：   
![result](https://static01.imgkr.com/temp/2dcfbe545a684ad1984bdece78b86030.png)  
2、字符串类型操作：  
![result](https://static01.imgkr.com/temp/be88bfd751a84d68901260d3406879e5.png)  
3、整数和浮点数类型数据操作：  
![result](https://static01.imgkr.com/temp/bdbcaf74328646daa81fb035acf77d84.png)  
4、list数据操作: (提供了从右或从左push或pop的方法，可以用作消息队列) 
![result](https://static01.imgkr.com/temp/a3af50e292c04a03aa603d8ebfdadf0f.png)   
5、set数据操作：  
![result](https://static01.imgkr.com/temp/0c53d69b177c4e72af9c116ca398c9bd.png)  
6、hash数据操作：  
![result](https://static01.imgkr.com/temp/906e706731734cc9882731e0300ec807.png)  
7、Zset数据操作：  
![result](https://static01.imgkr.com/temp/f455e391537e4964b19453b1abdc9ab0.png)  
8、对值为key的集合进行排序  
![result](https://static01.imgkr.com/temp/56ee976b3a3847a0ba8e6942d091cce4.png)  

## Redis的实例运用Jedis  
1、在maven项目中引入依赖  
```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```  
2、将数据从数据库加载到redis服务器上，并从redis上加载数据  
一、定义数据的类型：创建类：User  
```java
package com.wbg.mr.entity;

public class User {
    String uid;
    String userName;
    String passWord;
    String name;

    public User() {
    }
    public User(String uid, String userName, String passWord, String name) {
        this.uid = uid;
        this.userName = userName;
        this.passWord = passWord;
        this.name = name;
    }
    @Override
    public String toString() {
        return "User{" +
                "id='" + uid + '\'' +
                ", userName='" + userName + '\'' +
                ", passWord='" + passWord + '\'' +
                ", name='" + name + '\'' +
                '}';
    }
    public String getUid() {
        return uid;
    }
    public void setUid(String uid) {
        this.uid = uid;
    }
    public String getUserName() {
        return userName;
    }
    public void setUserName(String userName) {
        this.userName = userName;
    }
    public String getPassWrod() {
        return passWord;
    }
    public void setPassWrod(String passWord) {
        this.passWord = passWord;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```
二、创建从数据库加载数据的UserDao类  
```java
package com.wbg.mr.dao;

import com.wbg.mr.entity.User;
import redis.clients.jedis.Jedis;

import java.util.*;

public class UserDao {
    private static Jedis jedis;

    public UserDao(Jedis jedis) {
        this.jedis = jedis;
    }

    /**
     * 添加
     */
    public void addUser(User user) {
        //首先保存user-id
        jedis.sadd("useradd", "user-" + user.getUid());
        //-----添加数据----------
        Map<String, String> map = new HashMap<String, String>();
        map.put("uid", user.getUid());
        map.put("userName", user.getUserName());
        map.put("passWord", user.getPassWrod());
        map.put("name", user.getName());
        jedis.hmset("user-" + user.getUid(), map);
    }

    /**
     * 获取单个User
     *
     * @return
     */
    public List<String> getById(String id) {
        if (exists()) {
            return jedis.hmget("user-" + id, "id", "userName", "passWord", "name");
        }
        return null;
    }
    /**
     * 获取全部
     *
     * @return
     */
    public List<User> listAll() {
        List<User> list = new ArrayList<User>();
        User user = null;
        if (exists()) {
            for (String useradd : jedis.smembers("useradd")) {
                user = new User();
                List<String> lists = jedis.hmget(useradd, "id", "userName", "passWord", "name");
                user.setUid(lists.get(0));
                user.setUserName(lists.get(1));
                user.setPassWrod(lists.get(2));
                user.setName(lists.get(3));
                list.add(user);
            }
            return list;
        }
        return null;
    }

    //删除全部
    public boolean delAll() {
        if (exists()) {
            jedis.del("useradd");
            return true;
        }
        return false;
    }

    //判断是否存在
    public boolean exists() {
        return jedis.exists("useradd");
    }
}
```  
三、测试    
(一) 使用创建jedis或数据库连接池中获取jedis的方式测试
```java
public class Main {
    private static Jedis jedis =null;
    public static void main(String[] args) {
         //连接本地的 Redis 服务
         jedis = new Jedis(String ip,int port);
         //或创建数据库连接池，从数据库连接池中获取jedis 
        JedisPool jedisPool = new JedisPool(String ip,int port);  
        jedis = jedisPool.getResource();
        System.out.println("连接成功");
        //查看服务是否运行
        System.out.println("服务正在运行: "+jedis.ping());
        user();
    }
    public static void user(){
        UserDao user = new UserDao(jedis);
        user.delAll();
        user.addUser(new User("21","ldl","123456","刘地林"));
        user.addUser(new User("31","oyl","123456","欧一乐"));
        user.addUser(new User("41","tyj","123456","唐玉棋"));
        user.addUser(new User("51","cs","123456","陈胜"));
        user.addUser(new User("61","gsq","123456","郭世棋"));
        for (User user1 : user.listAll()) {
            System.out.println(user1);
        }
}
```  
(二)使用集群的方式测试(只需要将所有Jedis类型换成JedisCluster)  
