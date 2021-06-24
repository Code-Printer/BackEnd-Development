## Springboot  
### SpringBoot相比于Spring的优势  
1、快速整合第三方框架，比如redis，mybatis等等  
2、全部采用注解方式，没有繁琐的xml配置。  
3、内置http服务器，比如tomcat，jetty。不需要额外的去集成下载tomcat。
### SpringBoot与Spring的区别  
SpringBoot是Spring的扩展，它简化了Spring的依赖配置、MVC配置、模板引擎配置、安全配置、 打包和部署。使得开发、测试、部署变得更加方便。
### SpringBoot与SpringMVC的区别  
SpringMVC提供一种轻度耦合的方式开发web应用。  
SpringBoot实现了自动配置，无需xml文件手动配置，降低了项目搭建的复杂度。
### SpringBoot的常用注解及作用  
#### @SpringBootApplication
这个注解是Spring Boot最核心的注解，用在 Spring Boot的主类上，标识这是一个 Spring Boot 应用，用来开启 Spring Boot 的各项能力。实际上这个注解是@Configuration,@EnableAutoConfiguration,@ComponentScan三个注解的组合。由于这些注解一般都是一起使用，所以Spring Boot提供了一个统一的注解@SpringBootApplication。  
#### @EnableAutoConfiguration
允许 Spring Boot 自动配置注解，开启这个注解之后，Spring Boot 就能根据当前类路径下的包或者类来配置 Spring Bean。  
#### @ComponentScan  
组件扫描。让spring Boot扫描到Configuration类并把它加入到程序上下文。@ComponentScan注解默认就会装配标识了@Controller，@Service，@Repository，@Component注解的类到spring容器中。  
#### @Repository  
用于标注数据访问组件，即DAO层组件。使用@Repository注解可以确保DAO或者repositories提供异常转译(捕获异常后，输出另一个异常(作用在于捕获的如果是底层异常，输出该异常并不能准确表达意思，故需要输出高层可以表意的异常))，这个注解修饰的DAO或者repositories类会被ComponetScan发现并配置，不需要为它们在XML文件中配置项。  
#### @Service  
这个注解修饰的业务逻辑层的类会被ComponetScan发现并配置，不需要为它们在XML文件中配置项。  
#### @RestController  
这个注解修饰的控制层的类会被ComponetScan发现并配置。@RestController相当于@Controller+@ResponseBody，可以返回JSON，XML或自定义media Type数据内容到页面，但注解了@RestController视图解析器就不能解析jsp、html页面，故不能返回jsp、html页面。  
#### @Component  
泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。  
#### @AutoWired(spring定义的注解方式，只支持spring全家桶)  
把配置好的Bean拿来用，完成自动装配的工作。它可以对类成员变量、方法及构造函数进行标注。当加上（required=false）时，就算找不到bean也不报错。  
#### @Resource(name="name",type="type")(java定义的注解方式，扩展性强)  
与@AutoWired作用相同  
#### @RequestMapping  
 RequestMapping是一个用来设置请求地址映射的注解；用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。  

### SpringBoot开启事务(事务(ACID)：要么都执行，要么都不执行)  
1、在入口类使用注解@EnableTransactionManagement开启事务  
2、在访问数据库的service方法上添加注解@Transactional即可  

### SpringBoot的自动装配原理  
springboot的自动装配就是通过自定义实现的ImportSelector接口，从而导致项目启动时会自动将所有项目META-INF/spring.factories路径下的配置类注入到spring容器中，从而实现了自动装配。 






