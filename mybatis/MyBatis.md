# MyBatis  
mybatis是持久层的一个框架，不用写jdbc复杂的编码过程(直接可以在配置文件中写配置和读写操作)，就可以对数据库进行增删改查操作。
## MyBatis的操作步骤  
1、获取sqlSession对象  
2、创建一个操作数据库的接口(UserMapper)  
3、再创建一个映射配置文件（UserMapper.xml），实现该接口并配置读写操作方法或者使用注解的方法在上述接口的对应方法上注解实现读写操作。  
4、在myBatis.xml的核心配置文件中注册上一步的映射    
## Mybatis中的#{}与${}的区别    
 #{}，会将所有的参数值转化成字符串进行填充，会进行语句的预编译操作；
 ${}，不会进行参数的类型转换，直接参数值拼接，会引发注入问题。

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
2、工程的src/main/resources/目录下右键new → File；创建MyBatis的核心配置文件mybatis.xml (可任意起名)  
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
(2) 它一旦被创建就应该在运行期间一直存在。  
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
### 其他注意点
1、查询结果是Map类型
(1) 查询结果是一项封装成为一个Map对象时，resultType属性值为 ”map”  
(2) 查询结果是多项封装成为一个Map对象时，需要在Mapper接口的方法上使用注解
@MapKey(“xxx”)，将查询结果的xxx列作为key封装这个map，resultType属性值为value对应的类型(不再是map)    
### 使用注解进行开发(在对应的dao层逻辑操作类的方法上写注解进行操作)  
1、注解使用在Mapper接口的方法上，无需再使用UserMapper.xml文件  
```java
public interface UserMapper {
    
    @Select("select * from user limit #{startIndex}, #{pageSize}")  //这里必须用map数据结构进行填充
    List<User> getUserListById(Map<String, Integer> map);

}
```  
2、需要在核心配置文件mybatis.xml中绑定接口  
```xml
<mappers>
    <mapper class="com.qizegao.dao.UserMapper"/>
</mappers>
```    
3、测试  
```java
@Test
public void test01() {
    SqlSession sqlSession = MyBatisUtils.getSqlSession();
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

    //向map参数中赋值
    HashMap<String, Integer> map = new HashMap<String, Integer>();
    map.put("startIndex", 0);
    map.put("pageSize", 2);

    List<User> userListById = userMapper.getUserListById(map);
    sqlSession.close();
}
```
### 使用注解完成删除操作  
1、Mapper接口中添加注解  
```java
public interface UserMapper {
    @Delete("delete from user where id = #{uid}")
    int deleteUser(@Param("uid") int id);

    /**
     * 1. 方法上的注解还有@Insert()、@Update()两种，分别完成对应的操作
     * 2. 方法中的参数使用@Param注解表示给参数名起别名
     *
     * 注意： (1)参数有多个基本类型和String类型时需要使用@Param注解
     *       (2)参数如果只有一个基本类型时，可以不使用@Param注解，但尽量使用
     *       (3)类类型不需要使用@Param注解
     *
     */

}
```  
2、需要在核心配置文件mybatis.xml中绑定接口  
```xml
<mappers>
    <mapper class="com.qizegao.dao.UserMapper"/>
</mappers>
```  
3、测试
```java
public static SqlSession getSqlSession() {
    //可以在上面的MyBatisUtils工具类中设置sqlSession为自动提交，打开事务
    return sqlSessionFactory.openSession(true);
}
```
```java
@Test
public void test01() {
    SqlSession sqlSession = MyBatisUtils.getSqlSession();
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    int res = userMapper.deleteUser(3);
    //已经在工具类中设置为自动提交，故此处无需提交事务
    sqlSession.close();
}
```  
### 联合查询
有两张表t_key和t_lock，对应关系如下所示：  
![result](https://static01.imgkr.com/temp/a44e8d6a1a9145158871914f1197d3cf.png)  

1、级联属性的方式  
 (1)Key类和Lock类  
 ![result](https://static01.imgkr.com/temp/754edcce2614458a82f9ed57823810d6.png)  
(2)KeyMapper接口中声明  
```java
public Key getKeyById(Integer id);
```  
(3) KeyMapper.xml中编写  
```xml
 	<select id="getKeyById" resultMap="mykey">
 		select k.id, k.`keyname`, k.`lockid`, l.`id` lid, l.`lockName` 
		from t_key k
		left join t_lock l on k.`lockid`=l.`id`
		where k.`id`=#{id}
 	</select>

  	<resultMap id="mykey" type="com.qizegao.bean.Key" >
 		<id property="id" column="id"/>
 		<result property="keyName" column="keyname"/>
 		<result property="lock.id" column="lid"/>
 		<result property="lock.lockName" column="lockName"/>
 	</resultMap> 
```  
2. 使用association标签的方式
KeyMapper.xml中编写 (select标签的内容与上述一致)  
```xml
 	<resultMap id="mykey" type="com.qizegao.bean.Key" >
 		<id property="id" column="id"/>
 		<result property="keyName" column="keyname"/>
 		<!-- 使用association标签表示联合了一个对象 -->
 		<!-- javaType属性指定联合的对象类型 -->
 		<association property="lock" javaType="com.qizegao.bean.Lock">
 			<!-- 定义如何封装lock对象，property属性表示的是lock对象的属性 -->
 			<id property="id" column="lid"/>
 			<result property="lockName" column="lockName"/>
 		</association>
 	</resultMap>
```  
3. 使用collection标签的方式  
(1) Key类和Lock类  

![result](https://static01.imgkr.com/temp/a9f3f28ca94549beb8370cdfc02ec711.png)  

(2)LockMapper接口中声明(当查询的成员变量需求结果为一个集合时)
```java
public Lock getLockById(Integer id);
```  
(3) LockMapper.xml中声明  
```xml
 	<select id="getLockById" resultMap="mylock">
 		select l.*,k.id kid,k.`keyname`,k.`lockid` 
 		from t_lock l 
		left join t_key k on l.`id`=k.`lockid`
		where l.id=#{id}
 	</select>

 	<resultMap id="mylock" type="com.atguigu.bean.Lock">
 		<id property="id" column="id"/>
 		<result property="lockName" column="lockName"/>
 		<!-- 
 			使用collection标签表示联合了一个集合
 			ofType属性指定集合里面元素的类型
 		-->
 		<collection property="keys" ofType="com.atguigu.bean.Key">
 			<!-- 定义集合中的封装规则，property属性表示的是Key类中的属性 -->
 			<id property="id" column="kid"/>
 			<result property="keyName" column="keyname"/>
 		</collection>
 	</resultMap>
```  
### 动态sql  
1、创建一张数据库表t_teacher  
![result](https://static01.imgkr.com/temp/a733c50d44c24216826a6cb133d003c6.png)  

2、创建Teacher类  
```java
public class Teacher {

	private Integer id;
	private String name;
	private String course;
	private String address;
	private Date birth;
	//以及其余JavaBean结构(set、get、toString)
	
}
```  
3、TeacherMapper
```java
public List<Teacher> getTeacherByCondition(Teacher teacher);
```
4、TeacherMapper.xml中编写操作数据库操作  
```xml
	<resultMap id="teacherMap" type="com.qizegao.bean.Teacher">
		<id property="id" column="id" />
		<result property="address" column="address" />
		<result property="birth" column="birth_date" />
		<result property="course" column="class_name" />
		<result property="name" column="teacherName" />
	</resultMap>
	
	<select id="getTeacherByCondition" resultMap="teacherMap">
		
			select * from t_teacher
		
		<!-- trim标签用来截取字符串：
		
				(1) prefix属性为sql语句整体添加一个前后缀，一般设置为where，
				当没有条件符合where时，会自动去掉where前缀
				    
				(2) prefixOverrides属性去除每个if标签中指定的前缀，一般设置为and
					当某个and后面的条件不满足时，自动去掉and前缀
					
				(3) suffix和suffixOverrides属性使用与上述类似  		
		 -->

		<trim prefix="where" prefixOverrides="and">
		
			<!-- if标签的test属性写判断条件，
			     当满足判断条件时就将if标签体中的sql语句拼接到if标签体外的sql语句上
			     如 "id != null" 表示传入的JavaBean的id属性值不为null时才满足条件
		    -->
		
			<if test="id!=null">
				id > #{id}
			</if>

			<!-- &amp;表示&，&quot;表示" -->
			<if test="name!=null &amp;&amp; !name.equals(&quot;&quot;)">
				and teacherName like #{name} 
			</if>
			
			<if test="birth!=null">
			    <!-- &lt;表示< -->
				and birth_date &lt; #{birth} 
			</if>
		</trim>
	</select>
```    
5、测试类中编写  
```java
public class TeacherDaoTest {
    @Test
    public void test1(){
        SqlSession sqlSession = MyBatisUtils.getSqlSession();
        TeacherDao userMapper = sqlSession.getMapper(TeacherDao.class);
        Teacher teacher = new Teacher(2,"","","",null);//在此测试用例中设置id不为null，name不为null，birth为null
        List<Teacher> list = userMapper.getTeacherByCondition(teacher);
        System.out.println(list);
    }

}
```   
### 动态sql的其他标签  
1. foreach标签（在数据库操作的in语句可以使用）将集合中的数据一个一个取出来对比  
(1) TeacherMapper中编写  
```java
public List<Teacher> getTeacherByIdIn(@Param("ids")List<Integer> ids);
```  
(2) TeacherMapper.xml中编写  
```xml
	<select id="getTeacherByIdIn" resultMap="teacherMap">
		
		SELECT * FROM t_teacher WHERE id IN
		
		<!-- foreach标签用来遍历集合
				1. collection属性指定要遍历的集合 
				2. item属性为每次遍历出的元素起一个变量名
				3. separator属性定义遍历到的元素的分隔符
				4. open属性定义foreach标签中的sql语句以什么开始
				5. close属性定义foreach标签中的sql语句以什么结束
				6. index属性表示索引：
					(1) 如果遍历的是一个list: 
							index：指定的变量保存了当前索引 
							item：保存当前遍历的元素的值 
					(2) 如果遍历的是一个map： 
							index：指定的变量保存了当前遍历的元素的key 
							item：保存当前遍历的元素的值
		-->
		
		<if test="ids.size >0">
			<foreach collection="ids" item="id_item" separator="," open="("
				close=")">
				#{id_item} <!-- 取出遍历的元素的值 -->
			</foreach>
		</if>
	</select>
```
2. choose标签   
（1）TeacherDao中编写
```java
public List<Teacher> getTeacherByConditionChoose(Teacher teacher);
```  
(2) TeacherDao.xml中编写  
```xml
	<select id="getTeacherByConditionChoose" resultMap="teacherMap">
		select * from t_teacher
		
		<!-- sql语句不写where，使用where标签 -->
		
		<where> <!-- 相当于在sql语句补了一个where，且自动去除and或or -->

			<!-- choose标签是从上向下判断，
				 一旦满足了when条件，就将when标签中的sql语句添加到原sql语句后面
				 一旦满足了一个when标签就不会去判断其余的when标签，跳出choose标签
			 -->
			
			<choose>
				<when test="id!=null">
					id=#{id}
				</when>
				<when test="name!=null and !name.equals(&quot;&quot;)">
					teacherName=#{name}
				</when>
				<when test="birth!=null">
					birth_date = #{birth}
				</when>
				<otherwise>
					1=1 <!-- 永远为true -->
				</otherwise>
			</choose>
		</where>
	</select>
```  
3. set标签（修改操作时使用）  
(1) TeacherDao中编写  
```java
public int updateTeacher(Teacher teacher);
```  
(2)TeacherDao.xml中编写  
```xml
	<update id="updateTeacher">
		UPDATE t_teacher
		
		<!-- 使用set标签代替sql语句中的set
			 可以自动的去掉if标签中sql语句后面多余的逗号
		 -->
		
		<set>
			<if test="name!=null and !name.equals(&quot;&quot;)">
				teacherName=#{name},
			</if>
			<if test="address!=null and !address.equals(&quot;&quot;)">
				address=#{address},
			</if>
			<if test="birth!=null">
				birth_date=#{birth}
			</if>
		</set>
		<where> <!-- where标签代替sql语句手写where -->
			id=#{id}
		</where>
	</update>
```