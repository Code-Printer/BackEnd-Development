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
### 使用MyBatis完成CRUD  
1、查询  
```java
//UserMapper接口中添加方法
User getUserById(int id);


<!-- UserMapper.xml的mapper标签中添加标签 -->
<!-- parameterType属性表示id属性对应的方法参数的类型 -->
<select id="getUserById" parameterType="int" resultType="com.qizegao.pojo.User">
    select * from user where id = #{id}
</select>

//测试类中添加
@Test
public void test2() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    User user = userMapper.getUserById(1);
    System.out.println(user);
    //User{id=1, name='周杰伦', pwd='123456'}
    sqlSession.close();
}
```  
2、插入  
```java
//UserMapper接口中添加方法
int addUser(User user);

<!-- UserMapper.xml的mapper标签中添加标签 -->
<insert id="addUser" parameterType="com.qizegao.pojo.User">
    <!-- 对象中的属性可以通过#{}直接提取出来 -->
    insert into mybatis.user (id, name, pwd) values (#{id},#{name},#{pwd});
</insert>

@Test
public void test2() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    int res = userMapper.addUser(new User(7, "黎明", "33445566"));
    System.out.println("受影响的行数：" + res);//受影响的行数：1
    sqlSession.commit(); //增删改需要提交事务
    sqlSession.close();
}
```  
3、删除  
```java
//UserMapper类中添加方法
int deleteUser(int id);

<!-- UserMapper.xml的mapper标签中添加标签 -->
<delete id="deleteUser" parameterType="int">
    delete from user where id=#{id}
</delete>

public void test2() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    int i = mapper.deleteUser(7);
    System.out.println("受影响的行数：" + i); //受影响的行数：1
    sqlSession.commit(); //增删改需要提交事务
    sqlSession.close();
}
```  
4、修改  
```java
//UserMapper类中添加方法
int updateUser(User user);

<!-- UserMapper.xml的mapper标签中添加标签 -->
<update id="updateUser" parameterType="com.qizegao.pojo.User">
    update user set name=#{name}, pwd=#{pwd} where id=#{id}
</update>

@Test
public void test2() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    int i = mapper.updateUser(new User(7, "蔡虚坤", "78956"));
    System.out.println("受影响的行数:" + i); //受影响的行数:1
    sqlSession.commit(); //增删改需要提交事务
    sqlSession.close();
}
```  
### MyBatis配置解析
在MyBatis的核心配置文件mybatis.xml (任意起名)的configuration标签中写配置
1. 环境变量 (environments)  
MyBatis配置文件中可以配置多种环境，使用environment标签声明，但只可以选择其中一种环境，通过environments标签的default属性决定使用哪一种环境  
2. 事务管理器 (transactionManager)
默认使用JDBC，其余可去中文文档查看
3. 数据源 (dataSource)
默认使用POOLED，其余可去中文文档查看
```xml
<configuration>
    <!-- 以下配置了两个环境，default标签中写某一环境的id属性值从而去使用它 -->
    <environments default="development">
        <environment id="development"> <!-- id属性给此环境起一个名称 -->
            <transactionManager type="JDBC"/> <!-- 默认 -->
            <dataSource type="POOLED"> <!-- 默认 -->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>

        <!-- 第二个环境 -->
        <environment id="newEnvironment">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>   
</configuration>
```  
4、配置数据库连接 (properties文件)  
可以使用properties标签引用外部配置文件  
(1) src/main/resources目录下创建一个配置文件db.properties
```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/test?useSSL=false&useUnicode=true&characterEncoding=UTF-8
root=root
password=234414
```  
(2)mybatis.xml中编写  
```xml
<configuration>

    <properties resource="db.properties">
        <!-- properties标签中也可以声明key - value，与外部配置文件配合使用 -->
        <property name="username" value="root"/>
        <property name="password" value="234414"/>
    </properties>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <!-- xml文件中 "&" 需要转义成为 "&amp;" -->
                <property name="url" value="${url}"/>
                <property name="username" value="${root}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com.mrgao.dao/UserMapper.xml" />
    </mappers>
</configuration>
```  
注意：mybatis.xml文件中congratulation标签中的标签声明有顺序要求   
![result](https://static01.imgkr.com/temp/4ad871b36b464adaa84804533cb8db8c.png)  

### 给类类型起别名  
类类型别名存在的意义是减少配置时需要填写全类名的冗余，用法有两种 (在配置文件mybatis.xml中)  
(1) 给全类名起别名  
```xml
<typeAliases>
    <!-- 以后在Mapper.xml中使用此类型时无需再写全类名，直接写User即可 -->
    <typeAlias type="com.qizegao.pojo.User" alias="User"/>
</typeAliases>
```  
（2）扫描指定包下的类，某一类的别名默认是类名首字母小写  
```xml
<typeAliases>
    <!-- 此包下的User类的别名成为user -->
    <package name="com.qizegao.pojo"/>
</typeAliases>
```  
注意：可以在对应的类上加注解@Alias(“xxx”)，则类名的别名成为指定的xxx  
### 映射器 (mappers)
用来注册绑定的Mapper.xml文件，有三种方式。每个Mapper.xml文件都需要在MyBatis.xml的核心配置文件中注册  
(1) 使用mapper标签中的resources属性  
```xml
<mappers>
    <mapper resource="com/qizegao/dao/UserMapper.xml"></mapper>
</mappers>  
```  
（2）使用mapper标签中的class属性
```xml
<mappers>
    <!-- class属性中写Mapper.xml文件绑定的Mapper接口 -->
    <mapper class="com.qizegao.dao.UserMapper"></mapper>
    <!--
        1. 接口和其Mapper.xml配置文件必须同名
        2. 接口和其Mapper.xml配置文件必须在同一个包下
    -->
</mappers>
```  
（3） 使用package标签
```xml
<mappers>
    <!-- package标签可以扫描指定包下的Mapper.xml文件 -->
    <package name="com.qizegao.dao"/>
    <!--
        1. 接口和其Mapper.xml配置文件必须同名
        2. 接口和其Mapper.xml配置文件必须在同一个包下
    -->
</mappers>
```
### MyBatis的生命周期和作用域  
![result](https://static01.imgkr.com/temp/934b9c6af6e14ddc8ed768a06b3e38e8.png)  
1、SqlSessionFactoryBuilder
使用其创建了SqlSessionFactory之后，就不再需要它了  
2、SqlSessionFactory
(1) 可以理解为数据库连接池  
(2) 它一旦被创建就应该在运行期间一直存在，没有理由丢弃它或重新创建另一个实例  
3、SqlSession
(1) 可以理解为是连接到连接池的一个请求  
(2) 它的实例不是线程安全的，因此不能被共享  
(3) 使用完之后应该关闭，防止资源被占用  
### 日志工厂（打日志发现操作数据库时，底层自动开启事务和关闭事务autoCommit）  
一个数据库操作出现了异常，使用日志是最好的排错助手  
在mybatis.xml中的configuration标签中使用settings标签指明使用哪一种日志实现
常见的日志实现有两种：
(1) STDOUT_LOGGING (标准日志输出)
i. 配置文件中声明  
```xml
<settings>
    <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```  
(2) LOG4J的使用自行网上搜索  
### 八、其他注意点
1、\${}和#{}的区别  
MyBatis有两种取值方式：  
(1) #{属性名}：是预编译的方式，参数的位置都使用?替代，参数值都是预编译设置进去的；
比较安全，不会有sql注入问题  
(2) \${属性名}：直接和sql语句拼串，不安全
一般都使用#{}，在不支持预编译的位置要进行取值才使用${}
2、查询结果是Map类型
(1) 查询结果是一项封装成为一个Map对象时，resultType属性值为 ”map”  
(2) 查询结果是多项封装成为一个Map对象时，需要在Mapper接口的方法上使用注解
@MapKey(“xxx”)，将查询结果的xxx列作为key封装这个map，resultType属性值为value对应的类型(不再是map)  