# MyBatis  
mybatis是持久层的一个框架，不用写jdbc复杂的编码过程(直接可以在配置文件中写配置和读写操作)，就可以对数据库进行增删改查操作。
## MyBatis的操作步骤  
1、获取sqlSession对象  
2、创建一个操作数据库的接口  
3、在创建一个映射配置文件，实现该接口并配置读写操作方法  
4、在myBatis.xml的核心配置文件中注册上一步的映射  
## MyBatis程序实例  
1、IDEA中创建一个普通的Maven项目(POM工程)MyBatis，此工程的pom.xml中导入依赖  
```xml
<dependencies>

    <!-- 导入mysql驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.47</version>
    </dependency>

    <!-- 导入MyBatis框架 -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.6</version>
    </dependency>

    <!--导入Junit-->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<!-- 将src/main/java目录下的资源也打包 -->
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```  
2、工程的src/main/resources/目录下右键new → File；创建MyBatis的核心配置文件mybatis-config.xml (可任意起名)  
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--configuration核心配置文件-->
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <!-- xml文件中 "&" 需要转义成为 "&amp;" -->
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```  
3、子工程的src/main/java/目录下创建一个包com.qizegao.utils编写MyBatis工具类MybatisUtils.java  
```java
public class MybatisUtils {

    //1. 获取SqlSessionFactory接口的对象

    private static SqlSessionFactory sqlSessionFactory;

    static {
        try {
            //声明MyBatis的核心配置文件mybatis-config.xml的位置
            String resource = "mybatis-config.xml";
            InputStream resourceAsStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //2. 获取SqlSession接口的对象
    //SqlSession中包含了操作数据库的方法
    public static SqlSession getSqlSession() {
        return sqlSessionFactory.openSession();
    }
}
```  
4、子工程的src/main/java目录下创建一个包com.qizegao.pojo编写User类  
```java
public class User {
    private int id;
    private String name;
    private String pwd;
    //以及标准java bean的其余结构(set、get方法)
}
```  
5、子工程的src/main/java目录下创建一个包com.qizegao.dao编写UserMapper接口(与UserDao作用一致)  
```java
public interface UserMapper {
    List<User> getUserList();
}
```  
6、子工程的src/main/resources目录下创建com.qizegao.dao包包下创建配置文件UserMapper.xml (名字与UserMapper对应)，此文件相当于之前的UserDaoImpl  
```xml
<?xml version="1.0" encoding="UTF8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- namespace属性绑定一个对应的Dao/Mapper接口，写全类名 -->
<mapper namespace="com.qizegao.dao.UserMapper">

    <!-- select标签代表查询语句 -->
    <!-- id属性对应Mapper接口中要执行的方法名 -->
    <!-- resultType属性表示查询结果的类型，写全类名 -->
    <select id="getUserList" resultType="com.qizegao.pojo.User">
       select * from user
    </select>

</mapper>
```  
7、每一个Mapper.xml都需要在MyBatis的核心配置文件mybatis-config.xml中注册故在mybatis-config.xml中添加如下：  
```xml
<mappers>
    <mapper resource="com/qizegao/dao/UserMapper.xml"/>
</mappers>
```  
8、子工程的src/test/java目录下创建包com.qizegao.dao (测试包尽量与原包同名)包下创建测试类UserMapperTest  
```java
public class UserMapperTest {
    @Test
    public void test() {
        //1. 获取SqlSession对象
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        //2. 获取UserMapper对象，执行其中的方法
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        List<User> list = userMapper.getUserList();

        //3. 打印查询结果
        System.out.println(list);

        //4. 关闭sqlSession
        sqlSession.close();
    }
}
运行结果：成功获取到数据
```  
