# SpringBoot
SpringBoot相比Spring简化了配置，内置了许多框架，简化了初始的项目搭建和开发过程。  
# 微服务架构  
每个功能都是可以独立替换和升级的软件单元  

## @PropertySource注解加载指定的.properties配置文件  
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

## SpringBoot中的ConfigurationProperties注解给组件批量注入配置文件属性(与@Value不同的在于@Value是从配置文件中单个赋值的)  
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


## 第一章  
@SpringBootApplication，这个注解放在启动类上，spring容器会自动初始化一些配置信息，扫描bean，初始化bean。  
@RestController是基于Restful风格的spring控制类，与@Controller不同的是，@RestController返回的都是数据，例如常用的json数据，而@Controller不仅可以返回数据，还可以返回视图，如我## 们的jsp页面。  
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
## 第五章  程序生成数据库表并构建表中字段的类型  







