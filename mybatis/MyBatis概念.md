## MyBatis的懒加载(延迟加载(对数据的推迟加载))  
延迟加载的应用要求：关联对象的查询与主加载对象的查询必须是分别进行的select语句，不能是使用多表连接的select查询。必须是先执行主加载对象的select语句，再延迟执行对关联对象的select查询。    
mybatis仅支持关联对象association和关联集合对象collection的延迟加载，association是一对一，collection是一对多查询，在mybatis配置文件中可以配置lazyloadingEnable=true/false来开启和关闭懒加载。  
延迟加载的原理：使用CGLIB为目标对象建立代理对象，当调用目标对象的方法时进入到拦截器方法。(比如调用a.getB().getName()，拦截器方法invoke()发现a.getB()为null，会单独发送事先准备好的查询关联B对象的sql语句，把B查询出来然后调用a.setB(b)，也是a的对象的属性b就有值了，然后调用getName()，这就是延迟加载的原理。)  
## mybatis的执行器  
mybatis有三种基本的Executor执行器：
（1）SimpleExecutor  
每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象。 
（2）PauseExecutor  
执行update或select，以sql做为key查找Statement对象，存在就使用，不存在就创建，用完后，不关闭Statement对象，而且放置于Map内，供下一次使用。简言之，就是重复使用Statement对象。  
（3）BatchExecutor  
执行update，将所有sql通过addBatch()都添加到批处理中，等待统一执行executeBatch()，它缓存了多个Statement对象，每个Statement对象都是addBatch()完毕后，等待逐一执行executeBatch()批处理。与JDBC批处理相同。    
Mybatis中使用Executor执行器：在mybatis的配置文件中，可以指定默认的ExecutorType执行器类型；也可以手动给DefaultSqlSessionFactory的创建SqlSession的方法传递ExecutorType类型参数。  
## myBatis查询多个id、myBatis常用属性  
```java
//方法  
Page<UserPoJo>  getUserListByIds(@Param("ids") List<Integer> ids);
//Mapp文件配置
<!--根据id列表批量查询user-->
<select id="getUserListByIds" resultType="com.guor.UserPoJo">
    select * from student
    where id in
    <foreach collection="ids" item="userid" open="(" close=")" separator=",">
        #{userid}
    </foreach>
</select>
```
## mybatis的一级缓存、二级缓存(前提必须是在同一sqlSession对象下)  
1、一级缓存：指的是mybatis中sqlSession对象的缓存，当我们执行查询以后，查询的结果会同时存入sqlSession中，再次查询的时候，先去sqlSession中查询，有的话直接拿出，当sqlSession消失时，mybatis的一级缓存也就消失了，当调用sqlSession的修改、添加、删除、commit()、close()等方法时，会清空一级缓存。  
2、二级缓存：指的是mybatis中的sqlSessionFactory对象的缓存，由同一个sqlSessionFactory对象创建的sqlSession共享其缓存，但是其中缓存的是数据而不是对象。当命中二级缓存时，通过存储的数据构造成对象返回。查询数据的时候，查询的流程是二级缓存 > 一级缓存 > 数据库。  
3、如果开启了二级缓存，sqlSession进行close()后，才会把sqlSession一级缓存中的数据添加到二级缓存中，为了将缓存数据取出执行反序列化，还需要将要缓存的pojo实现Serializable接口，因为二级缓存数据存储介质多种多样，不一定只存在内存中，也可能存在硬盘中。  
4、mybatis框架主要是围绕sqlSessionFactory进行的，具体的步骤：  
定义一个configuration对象，其中包含数据源、事务、mapper文件资源以及影响数据库行为属性设置settings。  
通过配置对象，则可以创建一个sqlSessionFactoryBuilder对象。  
通过sqlSessionFactoryBuilder获得sqlSessionFactory实例。  
通过sqlSessionFactory实例创建qlSession实例，通过sqlSession对数据库进行操作。   
实例：  
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
"http://mybatis.org/dtd/mybatis-3-config.dtd">  
 
<configuration>   
    <!-- 加载类路径下的属性文件 -->  
    <properties resource="db.properties"/>  
 
    <!-- 设置类型别名 -->  
    <typeAliases>  
        <typeAlias type="cn.itcast.javaee.mybatis.app04.Student" alias="student"/>  
    </typeAliases>  
 
    <!-- 设置一个默认的连接环境信息 -->  
    <environments default="mysql_developer">  
 
        <!-- 连接环境信息，取一个任意唯一的名字 -->  
        <environment id="mysql_developer">  
            <!-- mybatis使用jdbc事务管理方式 -->  
            <transactionManager type="jdbc"/>  
            <!-- mybatis使用连接池方式来获取连接 -->  
            <dataSource type="pooled">  
                <!-- 配置与数据库交互的4个必要属性 -->  
                <property name="driver" value="${mysql.driver}"/>  
                <property name="url" value="${mysql.url}"/>  
                <property name="username" value="${mysql.username}"/>  
                <property name="password" value="${mysql.password}"/>  
            </dataSource>  
        </environment>  
 
        <!-- 连接环境信息，取一个任意唯一的名字 -->  
        <environment id="oracle_developer">  
            <!-- mybatis使用jdbc事务管理方式 -->  
            <transactionManager type="jdbc"/>  
            <!-- mybatis使用连接池方式来获取连接 -->  
            <dataSource type="pooled">  
                <!-- 配置与数据库交互的4个必要属性 -->  
                <property name="driver" value="${oracle.driver}"/>  
                <property name="url" value="${oracle.url}"/>  
                <property name="username" value="${oracle.username}"/>  
                <property name="password" value="${oracle.password}"/>  
            </dataSource>  
        </environment>  
    </environments>  
 
    <!-- 加载映射文件-->  
    <mappers>  
        <mapper resource="cn/itcast/javaee/mybatis/app14/StudentMapper.xml"/>  
    </mappers>  
 
</configuration>  
```    
```java
public class MyBatisTest {
 
    public static void main(String[] args) {
        try {
            //读取mybatis-config.xml文件
            InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
            //初始化mybatis,创建SqlSessionFactory类的实例
            SqlSessionFactory sqlSessionFactory =  new SqlSessionFactoryBuilder().build(resourceAsStream);
            //创建session实例
            SqlSession session = sqlSessionFactory.openSession();
            /*
             * 接下来在这里做很多事情,到目前为止,目的已经达到得到了SqlSession对象.通过调用SqlSession里面的方法,
             * 可以测试MyBatis和Dao层接口方法之间的正确性,当然也可以做别的很多事情,在这里就不列举了
             */
            //插入数据
            User user = new User();
            user.setC_password("123");
            user.setC_username("123");
            user.setC_salt("123");
            //第一个参数为方法的完全限定名:位置信息+映射文件当中的id
            session.insert("com.cn.dao.UserMapping.insertUserInformation", user);
            //提交事务
            session.commit();
            //关闭session
            session.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
``` 
## mybatis防止sql注入  
1、检查变量数据类型和格式  
2、过滤特殊符号  
3、绑定变量，使用预编译语句  