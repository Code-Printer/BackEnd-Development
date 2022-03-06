# SpringBoot+Mybatis+Mysql搭建web项目  
## 创建SpringBoot项目  
1、打开IDEA，File -> New -> Project，选择Spring Initializr，然后next。  
2、修改Ariifact，下面的Name、package会自动修改；Packaging有两种模式，一种是Jar，一种是War；因为springboot中自带了tomcat，因此可以将项目打成jar，直接运行；而我现有项目是部署到tomcat上，因此我需要打成war包；然后next。  
3、勾选项目需要的依赖，然后next ，进入下一页 ，设置project name，点击finish完成。  
4、可选择：在pom文件中添加mybatis-generator依赖，使用Mybatis-generator工具，根据配置的数据库表自动生成model、dao和mapper的工具   
```xml
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.2</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.3.1</version>
        </dependency>
```  
## http请求访问并渲染页面，返回json格式的信息  
1、配置数据源application.yml和创建响应的信息实体类Welcome、响应类Response、响应码ResponseCode  
```yml
spring:
  datasource:
    url: jdbc:sqlserver://localhost:3306;DatabaseName="OneStudent"
    username: root
    password: 123456
    driver-class-name: com.microsoft.sqlserver.jdbc.SQLServerDriver
 
  mvc:
    view:
      prefix: /WEB-INF/views/
      suffix: .jsp
```
2、在pom.xml中添加视图渲染依赖
```xml
 <!-- 使用 jsp 必要依赖 -->
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
        </dependency>
```
3、视图渲染，创建WelcomeController、welcome.jsp  
1、创建WelcomeControlle类
```java
@RestController
@RequestMapping("/welcome")
public class WelcomeController {
 
    /*
    * 访问 welcome.jsp 页面
    * @return
    * */
    @RequestMapping("welcomeIndex")
    public ModelAndView welcomeIndex(){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("welcome");
        mv.addObject("name","xx");
        return mv;
    }
 
    /*
    * 返回 json 字符串
    * @return
    * */
    @RequestMapping("getWelcomeInfo")
    @ResponseBody
    public Response getWelcomeInfo(){
 
        /*
        * 测试数据
        * */
        List<Welcome> welcomes = new ArrayList<>();
        Welcome w1 = new Welcome();
        w1.setId(1);
        w1.setName("xx1");
        w1.setAge(11);
        w1.setGender("女");
 
        Welcome w2 = new Welcome();
        w2.setId(2);
        w1.setName("xx2");
        w1.setAge(22);
        w1.setGender("男");
 
        welcomes.add(w1);
        welcomes.add(w2);
 
        Response response = new Response();
        response.setData(welcomes);
        response.setRetcode(ResponseCode.SUCCESS);
        response.setRetdesc("Success");
        return response;
    }
}
```
2、创建welcome.jsp页面  
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>视图渲染</title>
</head>
<body>
    您好，${name}
</body>
</html>
```
## 使用mybatis generator工具自动生成与数据库交互的代码  
用于为数据库表创建*mapper.xml、实体类和dao(定义操作数据库表的方法)文件  
1、在pom.xml的插件中添加mybatis generator 自动生成代码工具的插件  
```xml
<build>
        <plugins>
            <!-- mybatis generator 自动生成代码插件 -->
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.2</version>
                <configuration>
                    <configurationFile>src/main/resources/generator/generatorConfig.xml
                    </configurationFile>
                    <overwrite>true</overwrite>
                    <verbose>true</verbose>
                </configuration>
            </plugin>
        </plugins>
    </build>
```
2、上面pom.xml配置的pugin路径resources/generator文件夹下添加generatorConfig.xml  
```xml
<?xml version="1.0" encoding="UTF-8"?>
 
<!DOCTYPE generatorConfiguration
            PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
            "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!-- 数据库驱动：选择你的本地硬盘上面的数据库驱动包 -->
    <classPathEntry location="D:\package\mysql-connector-java-8.0.15\mysql-connector-java-8.0.15\mysql-connector-java-8.0.15.jar"/>
    <context id="DB2Tables"  targetRuntime="MyBatis3">
 
        <commentGenerator>
            <property name="suppressDate" value="false"/>
            <!-- 是否去除自动生成的注释 true:是 ;  false:否 -->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
 
        <!-- 数据库链接 URL，用户名、密码 -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/test"
                        userId="root"
                        password="root">
        </jdbcConnection>
 
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>
 
        <!-- 生成模型的包名和位置,生成model实体类文件位置 -->
        <javaModelGenerator targetPackage="com.example.demo.entity"
                            targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>
 
        <!-- 生成映射文件的包名和位置 -->
        <sqlMapGenerator targetPackage="mybatis"
                         targetProject="src/main/resources">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>
 
        <!-- 生成DAO的包名和位置,生成mapper接口文件位置 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.example.demo.mapper"
                             targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>
 
        <!-- 要生成的表 tableName是数据库中的表名或视图名 domainObjectName是实体类名;
         需要生成的实体类对应的表名，多个实体类复制多份该配置即可-->
        <table tableName="user_tb"
               domainObjectName="UserBinding"
               enableCountByExample="false"
               enableUpdateByExample="false"
               enableDeleteByExample="false"
               enableSelectByExample="false"
               selectByExampleQueryId="false">
        </table>
    </context>
</generatorConfiguration>
```
3、在maven中使用命令行mybatis-generator:generate -e，根据数据库中的表生产相关的类。    
| 在配置启动环境栏中点击Edit Configurations -> 添加 -> Maven  
![](https://static01.imgkr.com/temp/d76ef0688e7e4ec6b0e13c2fb76207ec.jfif)   
|| 执行以上结果：生成了相应文件。
![](https://static01.imgkr.com/temp/614cfa1941f747649d17247a3d2482d4.jfif)  
4、 在application.yml中添加mybatis的配置  
```yml
mybatis:
  mapper-locations: classpath*:mybatis/*Mapper.xml
  type-aliases-package: com.example.demo002.entity
```  
5、 在StudentBindingMapper.java中添加 @Repository("studentBindingMapper")注解才能使用@MapperScan扫描到。    
```java
@Repository("studentBindingMapper")
public interface StudentBindingMapper {
    int insert(StudentBinding record);
 
    int insertSelective(StudentBinding record);
}
```
6、 在SpringbootdemoApplication.java添加@MapperScan  
```java
@SpringBootApplication
@MapperScan("com.example.demo002.mapper")
public class Demo002Application {
 
    public static void main(String[] args) {
        SpringApplication.run(Demo002Application.class, args);
    }
 
}
```
## 添加service、controller层  
1、项目层级  
![](https://static01.imgkr.com/temp/452eebfb7fe64aecbdad642493b98c24.jfif)  
2、添加StudentBindingService,操作数据库的业务操作  
```java
public interface StudentBindingService {
    int deleteByPrimaryKey(Long id);
    int insert(StudentBinding record);
    int insertSelective(StudentBinding record);
    StudentBinding selectByPrimaryKey(Long id);
    int updateByPrimaryKeySelective(StudentBinding record);
    int updateByPrimaryKey(StudentBinding record);
    void validTransaction(Long id);
    List<StudentBinding> getStudentBindByQuery(StudentBinding record);
}
```
3、添加StudentBindingServiceImpl，调用mybatis操作数据库的操作对象    
```java
@Service(value = "studentBindingService")
public class StudentBindingServiceImpl implements StudentBindingService {
 
    @Autowired
    private StudentBindingMapper studentBindingMapper;  //mybatis生成的操作数据库的接口
 
    @Override
    public int deleteByPrimaryKey(Long id){
        return studentBindingMapper.deleteByPrimaryKey(id);
    }
 
    @Override
    public int insert(StudentBinding record){
        return studentBindingMapper.insert(record);
    }
 
    @Override
    public int insertSelective(StudentBinding record){
        return studentBindingMapper.insertSelective(record);
    }
 
    @Override
    public StudentBinding selectByPrimaryKey(Long id){
        return studentBindingMapper.selectByPrimaryKey(id);
    }
 
    @Override
    public int updateByPrimaryKeySelective(StudentBinding record){
        return studentBindingMapper.updateByPrimaryKeySelective(record);
    }
 
    @Override
    public int updateByPrimaryKey(StudentBinding record){
        return studentBindingMapper.updateByPrimaryKey(record);
    }
 
    @Override
    @Transactional
    public void validTransaction(Long id){
        // 删除之后，插入该id的数据
        studentBindingMapper.deleteByPrimaryKey(id);
 
        StudentBinding record = new StudentBinding();
        record.setId(id);
        studentBindingMapper.insertSelective(record);
    }
 
    @Override
    public List<StudentBinding> getStudentBindByQuery(
            StudentBinding record){
        return studentBindingMapper.getStudentBindByQuery(record);
    }
}
```
4、新增StudentBindingController，定义后端请求接口和响应数据  
```java
@Controller
@RequestMapping(value = "/studentBind")
public class StudentBindingController {
 
    @Autowired
    private StudentBindingService studentBindingService;
 
    /*
    * 根据请求参数，删除绑定学生信息
    * @param id
    * @retutn
    * */
    @RequestMapping("deleteByPrimaryKey")
    @ResponseBody
    public Response deleteByPrimaryKey(Long id){
        Response response = new Response();
        if(id==null){
            response.setRetcode(ResponseCode.PARAMARTER_ERROR);
            response.setRetdesc("参数错误");
            return response;
        }
        try{
            studentBindingService.deleteByPrimaryKey(id);
            response.setRetcode(ResponseCode.SUCCESS);
            response.setRetdesc("删除成功");
        }catch (Exception e){
            e.printStackTrace();
            response.setRetcode(ResponseCode.FAILED);
            response.setRetdesc("删除异常");
        }
        return response;
    }
 
    /*
    * 根据请求参数，添加绑定学生信息
    * @param record
    * @return
    * */
    @RequestMapping("insertSelective")
    @ResponseBody
    public Response insertSelective(StudentBinding record){
        Response response = new Response();
        if(record==null){
            response.setRetcode(ResponseCode.PARAMARTER_ERROR);
            response.setRetdesc("参数错误");
            return response;
        }
 
        try{
            studentBindingService.insertSelective(record);
            response.setRetcode(ResponseCode.SUCCESS);
            response.setRetdesc("添加成功");
        }catch (Exception e){
            e.printStackTrace();
            response.setRetcode(ResponseCode.FAILED);
            response.setRetdesc("添加异常");
        }
        return response;
    }
 
    /*
    * 根据请求参数，查询绑定学生信息
    * @param id
    * @return
    * */
    @RequestMapping("selectByPrimaryKey")
    @ResponseBody
    public Response selectByPrimaryKey(Long id){
 
        Response response = new Response();
        if(id==null){
            response.setRetcode(ResponseCode.PARAMARTER_ERROR);
            response.setRetdesc("参数错误");
            return response;
        }
 
        try{
            StudentBinding studentBinding = studentBindingService.selectByPrimaryKey(id);
            response.setData(studentBinding);
            response.setRetcode(ResponseCode.SUCCESS);
            response.setRetdesc("查询成功");
        }catch (Exception e){
            e.printStackTrace();
            response.setRetcode(ResponseCode.FAILED);
            response.setRetdesc("查询异常");
        }
        return response;
    }
 
    /*
    * 验证@Transaction注解是否好用
    * @param id
    * @return
    * */
    @RequestMapping("validTransaction")
    @ResponseBody
    public Response validTransaction(Long id){
        Response response = new Response();
        if(id==null){
            response.setRetcode(ResponseCode.PARAMARTER_ERROR);
            response.setRetdesc("参数错误");
            return response;
        }
 
        try{
            studentBindingService.validTransaction(id);
            response.setRetcode(ResponseCode.SUCCESS);
            response.setRetdesc("添加成功");
        }catch (Exception e){
            e.printStackTrace();
            response.setRetcode(ResponseCode.FAILED);
            response.setRetdesc("添加异常");
        }
        return response;
 
    }
 
    /*
    * 跳转到上传文件页面
    * @return
    * */
    @RequestMapping("multipartIndex")
    public String multipartIndex(){
        return "multipart-index";
    }
 
    /*
    * 上传文件到指定目录
    * @param file
    * @return
    * */
    @RequestMapping("/upload")
    @ResponseBody
    public Response upload(@RequestParam("file") MultipartFile file){
        Response response = new Response();
        if(file.isEmpty()){
            response.setRetcode(ResponseCode.PARAMARTER_ERROR);
            response.setRetdesc("参数错误");
            return response;
        }
 
        try{
            String filePath = "D:\\ceshi\\upload";
            File dir = new File(filePath);
            if(!dir.isDirectory()){
                dir.mkdir();
            }
            String fileOriginalName = file.getOriginalFilename();
            File writeFile = new File(filePath + fileOriginalName);
            // 文件写入磁盘
            file.transferTo(writeFile);
 
            response.setRetcode(ResponseCode.SUCCESS);
            response.setRetdesc("上传成功");
        }catch (Exception e){
            e.printStackTrace();
            response.setRetcode(ResponseCode.FAILED);
            response.setRetdesc("上传失败");
        }
        return response;
    }
}
```
重启项目之后，就可以访问各个接口。  
## SpringBoot项目打包部署到tomcat上   
1、选择打包成jar包或war包：
jar包：将项目看成一整个拼图，引入的 jar包 就是一个拼块，在项目中引入的依赖就是jar包，里面是编译后的class文件。    
war包：在 javaweb中通常都是将项目打包成war包再进行部署，里面包括写的代码编译成的class文件，依赖的包，配置文件，所有的页面文件。一个war包可以理解为一个web项目。  
设置Springboot项目的打包格式可以在pom.xml中设置  
```xml
    <groupId>com.xian</groupId>
    <artifactId>project</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging>  //打包的格式在这里设置
    <name>project</name>
    <description>Demo project for Spring Boot</description>
```
2、在pom.xml里排除自带tomcat插件，有两种方法：  
第一种，在spring-boot-starter-web这个依赖里面新增
<exclusions>包裹的部分    
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```
第二种    
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```
3、将项目的启动类Application.java继承SpringBootServletInitializer并重写configure方法，注意将下面两个BootApplication替换成自己的启动类名      
```java
@SpringBootApplication
public class BootApplication extends SpringBootServletInitializer {
    public static void main(String[] args) {
        SpringApplication.run(BootdoApplication.class, args);
    }
 
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(BootApplication.class);
    }
 
}
```
4、修改打包后的名称，可以在pom.xml修改      
```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
        <finalName>项目名</finalName>
    </build>
```
5、Edit Configuration -> Maven -> 添加maven的package命令 ->执行package命令 -> 复制war包到 -> tomcat的webapps文件下 ->修改war包的名字 -> tomcat bin -> 打开startup.bat，打开tomcat容器  
添加maven的package命令：
![](https://static01.imgkr.com/temp/51665e2143a84e4eb424cff341891a5c.jfif)  
修改war包名字：  
![](https://static01.imgkr.com/temp/3d193c7b3ce94892a28511de4a2b58d0.jfif)  
6、访问注意事项，在IEDA中如果设置了端口号，例如访问地址http://localhost:8081/list，部署到tomcat中就变成了默认的8080端口，还要加项目名，即http://localhost:8080/war包的名称/list