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







