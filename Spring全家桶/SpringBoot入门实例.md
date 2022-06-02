# SpringBoot
SpringBoot相比Spring简化了配置，内置了许多框架，简化了初始的项目搭建和开发过程。  
# 微服务架构  
每个功能都是可以独立替换和升级的软件单元  

### @PropertySource注解加载指定的.properties配置文件  
1、新建一个person.properties文件
```java
person.last-name=李四
```  
2、在Bean组件中加入@PropertySource注解  
```java
@PropertySource(value = {"classpath:person.properties"})
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String lastName;
    public String getLastName(){
        return this.lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
}
```  
3、使用测试  
```java
@Controller
public class HelloController {
    @Autowired
    public Person person;

    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return person.getLastName();
    }

}
```

### SpringBoot中的ConfigurationProperties注解给组件批量注入配置文件属性(与@Value不同的在于@Value是从配置文件中单个赋值的)  
该注解有一个prefix属性，通过指定的前缀，绑定配置文件中的相应配置。类的属性名称必须与配置文件属性的名称相同；类的字段必须有公共 setter 方法。  
使用实例  
1、application.yml 配置文件  
```java
person:
  age: 18
  boss: false
  birth: 2017/12/12
  maps: {k1: v1,k2: 12}
  lists:
   - lisi
   - zhaoliu
  dog:
    name: wangwang
    age: 2
  last-name: wanghuahua
```  
2、配置文件的每个属性值，映射到bean组件中  
```java
@Component
@ConfigurationProperties(prefix = "person")  //表明使用的是配置文件中的person这个属性  
public class Person {
    //或者使用@Value给lastName赋值  
    //@Value("${person.last-name}")
    public String lastName;
    private Integer age;
    private Boolean boss;
    private Map<String, Object> maps;
    private List<Object> lists;

    public String getLastName(){
        return this.lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

}
```  
3、使用   
```java
@Controller
public class HelloController {
    @Autowired
    public Person person;

    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        System.out.println(person.getLastName());
        return person.getLastName();
    }

}
``` 

### SpringBoot给容器添加组件的三种方式  
1、通过手写xml中配置bean标签，来给容器添加类对象组件   
2、使用自动扫描@ComponScan搭配(@Controller、@Service、@Repository@Component)  
3、使用给类上添加@Configuration和给有返回类对象的方法上添加@Bean注解来给容器添加组件对象  
```java
@Configuration
public class MyAppConfig {

    //将方法的返回值添加到容器中；容器这个组件id就是方法名
    @Bean
    public HelloService helloService01(){
        System.out.println("配置类给容器添加了HelloService组件");
        return new HelloService();
    }
}
```  

### SpringBoot项目使用占位符(${})和默认值（:）给配置文件的属性赋初值  
主要可以应用在数据库的参数配置上(路径、用户名和密码)：数据库配置在本地配置的参数是本地的，部署时候需要命令替换  
```properties
person.age=${random.int}  //{}这个是字符串拼接时使用
person.boss=false
person.last-name=张三${random.uuid}
person.maps.k1=v1
person.maps.k2=v2
person.lists=a,b,c
person.dog.name=${person.last-name:wanghuahu} //初始值是wanghuahu
person.dog.age=15
```  
### @Conditional注解  
作用：必须满足@Conditional指定的条件，才注入到容器中。

| @Conditional派生注解                | 作用（判断是否满足当前指定条件）               |
| ------------------------------- | ------------------------------ |
| @ConditionalOnJava              | 系统的java版本是否符合要求                |
| @ConditionalOnBean              | 容器中存在指定Bean                    |
| @ConditionalOnMissBean          | 容器中不存在指定Bean                   |
| @ConditionalOnExpression        | 满足spEL表达式                      |
| @ConditionalOnClass             | 系统中有指定的类                       |
| @ConditionalOnMissClass         | 系统中没有指定的类                      |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean,或者这个Bean是首选Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                |
| @ConditionalOnResource          | 类路径下是否存在指定的资源文件                |
| @ConditionalOnWebApplication    | 当前是web环境                       |
| @ConditionalOnNotWebApplication | 当前不是web环境                      |
| @ConditionalOnJndi              | JNDI存在指定项  
### 自动配置报告  
在控制台打印自动配置的信息，可以通过在SpringBoot项目的全局配置文件中配置debug=true属性，就可以打印自动配置报告。  
```java
Positive matches:（启动的，匹配成功的）
-----------------

   CodecsAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.http.codec.CodecConfigurer'; @ConditionalOnMissingClass did not find unwanted class (OnClassCondition)
        ......
        
 Negative matches:（没有启动的，没有匹配成功的）
-----------------

   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'javax.jms.ConnectionFactory', 'org.apache.activemq.ActiveMQConnectionFactory' (OnClassCondition)
.....
```  
## 日志配置  
SpringBoot默认使用LogBack日志系统，日志会记录程序中的Error、warn、info(默认使用该级别)、debug、 trace级别的日志信息(日志级别是逐渐降低的，如果日志级别设置为INFO，则低于INFO级别的日志都看不到)，可以用于程序员针对不同情况快速定位程序位置。可以使用LOG.error()(warn、info、debug、trace)方法打印程序中出现的错误信息。  
1、在SpringBoot项目中添加LogBack日志依赖（一般web项目中在启动器中已经包含该依赖，故可以不添加）  
```xml
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-logging</artifactId>
```
2、项目中引用日志实例（使用INFO级别打印日志至控制台）:新建一个配置类LogConfig，注入一个Bean，并在方法中打印日志。    
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
@Configuration
public class LogConfig {
    private static final Logger LOG = LoggerFactory.getLogger(LogConfig.class);

    @Bean
 public Person logMethod() {
   try{
     LOG.info("==========print log==========");
   }catch(Exception e){
      LOG.error("==========print log==========");
   }
        
       return new Person();
    }
}  

//或使用注解方式
@Slf4j
@Configuration
public class LogConfig {

    @Bean
 public Person logMethod() {
   try{
     log.info("==========print log==========");//类上使用了注解@Slf4j，就可以直接使用log变量
   }catch(Exception e){
      log.error("==========print log==========");
   }
        
       return new Person();
    }
}
```  
控制端输出的日志信息(时间日期、日志级别、进程ID、分隔符、线程名、Logger名(一般是类名，便于定位)、日志内容)  
![result](https://static01.imgkr.com/temp/cfa0840ca71e49ebbd882b5e3f2e6756.png)  
3、将日志信息存储在文件(用于线上调试时查看程序bug信息)   
在maven项目的resources目录下的application.yml文件中添加logging属性,就可以打印日志信息到指定文件。  
```yml
logging:
  file:
    name: springbootdemo.log
    path: C:\Users\mrgao\IdeaProjects\test2\
```
4、设置日志级别  
日志级别总共有TRACE < DEBUG < INFO < WARN < ERROR < FATAL ，且级别是逐渐降低的，如果日志级别设置为INFO，则意味TRACE和DEBUG级别的日志都看不到，可以在参数文件配置文件中修改日志的打印级别。设置日志打印级别可以是项目的所有日志(logging.level.root)，也可以设置包范围的日志打印级别(logging.level.包名的全名)。  
```yml  
logging:
  level:
    root: INFO  //设置全项目使用的打印日志级别是INFO
    com.jackie.springbootdemo.config: WARN  //com.jackie.springbootdemo.config包下使用的打印日志级别是WARN
```  
5、定义自己的日志信息在文件和控制台的打印格式：  
在resources目录下的application.yml文件中添加如下参数  
```yml
logging:
  pattern:
      file: "%d{yyyy/MM/dd-HH:mm} [%thread] %-5level %logger- %msg%n"
      console: "%d{yyyy/MM/dd-HH:mm:ss} [%thread] %-5level %logger- %msg%n"
```  
## Springboot框架的Web开发(Springboot项目只需要将项目打包成jar包，使用java -jar xxx运行项目。)  
使用Springboot框架开发web项目有别与传统的web项目(不使用Springboot框架开发的)开发，使用Springboot框架开发的web项目是没有WEB-INF目录，且静态页面是不放在WEB-INF同目录下的，Springboot框架开发的web项目的静态资源是放在resource目录下的static目录下，动态资源或模板是放在template目录下的。  
前后端分离开发：前后端是完全解耦的，后端将功能写成rest API形式，前端可以使用自己的框架，只需要调用后端api，进行数据回显就行了。  
### SpringBoot的web依赖配置  
SpringBoot极大地简化了Spring应用从搭建到开发的过程，做到了开箱即用的方式。开发者想要快速开发web应用，只要在pom.xml加入了spring-boot-starter-web依赖即可。    
spring-boot-starter-web的组成如下表：  
1、spring-boot-starter  核心包，包括了自动化配置支持、日志、YAML 文件解析的支持等。  
2、spring-boot-starter-json 读写 JSON 包  
3、spring-boot-starter-tomcat Tomcat 嵌入式 Servlet 容器包  
4、hibernate-validator Hibernate 框架提供的验证包  
5、spring-web Spring 框架的 Web 包  
6、spring-webmvc Spring 框架的 Web MVC 包  
### servlet相关配置  
在resource/application.properties文件中添加以下配置更改服务监听端口和服务环境路径：  
```properties  
#端口号
server.port=8081
#服务环境路径
server.servlet.context-path=/demo   
#访问项目路径：http://localhost:端口号/demo/
```   
### SpringBoot设置默认访问页面  
新建一个类WebConfigurer实现WebMvcConfigurer类重写里面的addViewControllers方法  
```java
@Configuration
public class WebConfigurer implements WebMvcConfigurer{
   @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("forward:index.html");
        registry.setOrder(Ordered.HIGHEST_PRECEDENCE);
    }
}
```  

## 第一章  
@SpringBootApplication，这个注解放在程序的入口启动类上，spring容器会自动初始化一些配置信息，扫描bean，初始化bean。  
@RestController是基于Restful风格的spring控制类，与@Controller不同的是，@RestController只能返回数据，例如常用的json数据，而@Controller不仅可以返回数据，还可以返回视图，比如我们的jsp页面。  
@RequestMapping定义的是请求的映射路径。  
@GetMapping与@RestController是一块的，表示这是个get请求，与@RequestMapping(method = RequestMethod.GET)是一个意思。   
## 第二章  
Freemarker模板引擎：将页面渲染放在客户端进行，当前后端不分离时，页面显示的效率比较好；当前后端分离时，使用模板引擎作用不大。    
## 第三章 添加Druid数据库连接池源  
1、在xml文件下  
```xml
 <properties>
    <druid.spring.boot.version>1.1.21</druid.spring.boot.version>
  </properties>
//添加druid依赖
   <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid-spring-boot-starter</artifactId>
      <version>${druid.spring.boot.version}</version>
    </dependency>
//添加JDBC依赖
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
//数据库驱动
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
    </dependency>
```     
2、在resources目录下添加application.yml，并填写数据库的参数配置  
```yml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/spring_boot_study?useUnicode=true&characterEncoding=UTF8&useSSL=false
    username: root
    password: root
```  
## 第四章  添加jpa支持
jpa：是官方提供的ORM规范，将java对象以数据库记录的形式存储到数据库中。Hibernate是基于JPA实现的，JPA用起来简单，碰到复杂的情形，我们一样可以使用原生sql来解决。  
1、在pom.xml文件添加依赖  
```xml
 <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
```  
2、在项目启动类上添加扫描Repository注解  
```java
@EntityScan(basePackages = {"org.example"})
@EnableJpaRepositories(basePackages = {"org.example"})
public class Application {

}
```
EntityScan表示扫描带有Entity注解的JPA实体，EnableJpaRepositories扫描带有Repository注解的DAO类。如果basePackages为空，则会将启动类所在的包路径作为根路径。 







