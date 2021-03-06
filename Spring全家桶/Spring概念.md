# Spring(简化开发)  
## Spring框架的好处   
1、IOC(依赖注入，有依赖关系的对象从主动创建到从容器中注入的方式)  
2、AOP(切入方法后进行日志打印)  
3、提供了jUnit4的支持，可以通过注解(test)测试程序  
4、可以集成其他框架(mybatis)
## Spring创建bean的流程：  
1、通过bean工厂中获取对应bean(getBean方法)  
2、如果没有对应bean，则通过反射的方式创建bean实例(createBeanInstance方法)  
3、再填充该bean实例属性(populateBean方法)  
4、执行该bean的初始化方法(使用initializeBean方法)    
## Spring中beaen的生命周期  
1、通过构造器或工厂方法创建bean实例(默认创建的是单例对象)；  
2、为bean的属性赋值；  
3、调用bean的初始化方法；  
4、使用bean；  
5、当容器关闭时，调用bean的销毁方法；  
## Spring的三级缓存
1、一级缓存：存放完整创建的单例对象   
2、二级缓存：存放半成品的bean或代理对象，用于解决循环依赖(简单理解：就是在a类中引用了b类对象，在b类中引用了a类对象。故创建a的bean时，会需要创建b的bean，但填充b的属性时，a的bean是一个半成品，需要放在二级缓存中。)  
3、三级缓存(有一个lambda表达式，是将为未初始化完全或填不完全的bean转化成其代理对象)：存放单例工厂，用于生成动态代理对象，返回该bean的代理对象(简单理解：当当前bean被其他bean引用时，如果自己未实例化完全，是一个半成品bean，则需要使用其动态代理，返回自己的动态代理对象。)    
### 循环依赖问题
Spring框架中在A类中引用B类，在B类中引用A类。此时如果获取A的bean对象时，先会从bean容器中寻找A的bean对象，发现不存在则会使用createBeanInstance创建A的bean对象，创建完成后需要去填充A的bean组件属性，此时发现A中的属性是B的对象，会去bean仓库中寻找B的bean，发现B的bean不存在，就会去创建B的bean，当填充B的属性时，发现B中的属性为A的对象，此时A的bean为一个半成品，故需要使用二级缓存，将半成品A的bean存放在二级缓存中，以便B进行属性填充时使用。  
![result](https://static01.imgkr.com/temp/182e6ed55c974c3fb1fe9a4f15873623.png)
### 动态代理问题   
动态代理是指在一个类中，如果调用另一个类，而这个类并未在bean工厂创建完整的bean，此时需要使用这个另一个类的代理对象。   
## Spring的自动装配原理  
自动装配：在springboot中使用少量注解和配置即可使用引入的依赖功能  
Spring Boot 通过@EnableAutoConfiguration开启自动装配，再通过SpringFactoriesLoader最终加载META-INF/spring.factories中的自动配置类，实现自动装配。(自动配置类是通过@Conditional按需加载的配置类的)，想要其生效必须引入spring-boot-starter-xxx包实现起步依赖。  
## BeanFactory和ApplicationContext的区别  
BeanFactory是一个负责生产和管理bean的工厂接口，提供一个SpringIOC的容器规范，只有在获取时才会创建该bean对象。
ApplicationContext是BeanFactory的子接口，扩展了很多高级特性(同时加载多个配置文件、统一的资源访问方式)，每次容器启动时就会创建所有的对象。可以以声明的方式创建。   
## BeanFactory与FactoryBean的区别  
BeanFactory是一个负责生产和管理bean的工厂接口，提供一个SpringIOC容器规范。    
FactoryBean是一个bean，Spring提供创建bean实例的另一种方式(通过重写getObject方法改写获取bean的实例对象内容)。  
## BeanFactoryPostProcessor与BeanPostProcessor的区别  
BeanFactoryPostProcessor：在**实例化bean之前(在调用bean的构造函数创建对象之前)**，可以拿到bean的定义信息并修改它，比如修改bean的属性值或向bean容器中注入其他bean的定义信息。**处理对象的是整个bean工厂。**  
BeanPostProcessor：在**实例化bean之后，初始化bean之前或之后(此时bean已经完成了实例化)**，修改bean的实例化信息，比如生成一个动态代理对象。**处理对象是单个bean。**
## Spring的类中引用类型属性的赋值装配方式  
1、no：默认的方式是不进行自动装配，通过显式设置bean标签的ref属性来进行装配  
2、byName：通过参数名自动装配，Spring 容器在配置文件中将bean的autowire属性被设置成byname，之后容器试图装配和该bean的属性具有相同名字的bean。  
3、bytype：通过参数类型自动装配，Spring 容器在配置文件中将bean的autowire属性被设置成byType，之后容器试图装配和该bean属性具有相同类型的bean。如果有多个bean符合条件，则抛出错误。
4、constructor：Spring容器在配置文件中将bean的autowire属性被设置成constructor(要求类的定义要有构造器，使用构造器对类中属性进行赋值)，之后容器试图装配和该bean属性类型相同的bean。如果没有确定的带参数的构造器参数类型bean，将会抛出异常。  
5、autodetect：Spring容器在配置文件中将bean的autowire属性被设置成autodetect，该bean标签在装配属性时首先尝试使用constructor的方式来自动装配，如果无法工作，则使用byType方式。  
## JavaConfig(注解配置，可以理解为xml中的beans标签)  
JavaConfig是Spring3.0新增的概念，使用注解的方式替换了xml文件。javaConfig结合了xml的解耦和java编译时检查的优点：    
@Configuration，表示这个类是配置类，可以在这个类中使用@bean标签，定义bean对象；  
@ComponentScan，相当于xml的<context:componentScan basepackage=>，组件扫描；  
@Bean，相当于xml中的bean标签；  
@EnableWebMvc，相当于xml的<mvc:annotation-driven>；  
@ImportResource，引入新的spring配置文件.xml  
@PropertySource，用于读取properties配置文件；  
@Profile，一般用于多环境加载对应配置场景，激活时可用@ActiveProfile("环境名")注解；  

## Spring中@Import注解和@ImportResource的区别  
@Import注解是在该类中引入带有@Configuration类的bean组件。
@ImportResource是引入spring配置文件.xml。应用场景：有多个配置xml文件，然后需要整合到beans.xml中，就可以使用ImportResource实现整合引入其他的xml配置bean组件 
```java

//使用@ImportResource
@Configuration
@ImportResource(value = "beans-another.xml")  //在该类中引入beans-another.xml文件中的全部bean组件
public class SpringConfiguration {
    @Bean(name = "stu",autowire = Autowire.BY_TYPE)
    @Scope(value = "singleton")
    public Student student(){
        return new Student(11,"jack",22);
    }

//使用@import  
@Configuration
public class BookConfiguration {
 
    @Bean
    public Book book(){
        return new Book();
    }
}
 
// 替换上文的@ImportResource方式为@Import
@Configuration
@Import(value = BookConfiguration.class)  //引入类为BookConfiguration中的bean组件，该类需要进行@Configuration注解才可以
//@ImportResource(value = "beans-another.xml")
public class SpringConfiguration {
 
    @Bean(name = "stu",autowire = Autowire.BY_TYPE)
    @Scope(value = "singleton")
    public Student student(){
        return new Student(11,"jack",22);
    }
}
 
// 测试
AnnotationConfigApplicationContext applicationContext
                = new AnnotationConfigApplicationContext(SpringConfiguration.class);
Book book = (Book) applicationContext.getBean("book");
System.out.println(book);

```

## Spring中的@profile注解的作用  
@profile注解创建一个特定环境，只有激活该特定环境才能注入相应的bean组件。最常见的就是使用不同的DataSource了。  
说明代码：
1、MoveFactor.interface
```java
package com.xueyou.demo;
public interface MoveFactor {
    void speak();
}
```  
2、创建Chinese、English、German实现MoveFactor接口  
```java
//Chinese
 
 
@Configuration
@Profile(value = "dev")//该类只有当环境设置为dev才使用
@Component
public class Chinese implements MoveFactor {
    @Override
    public void speak() {
        System.out.println("我是中国人");
    }
}


//English
 
@Component
@Profile("qa")//该类只有当环境为qa才使用
public class English implements MoveFactor{
    @Override
    public void speak() {
        System.out.println("i am an English");
    }
}

//German  
@Component
@Profile("prod")   //该类只有当环境为prod才使用
public class German implements MoveFactor{
    @Override
    public void speak() {
        System.out.println("i am a German");
    }
}

```  
3、创建一个Person类，使用MoveFactor类中的speak功能  
```java
@Component
public class Person {

    @Autowired
    private MoveFactor moveFactor;

    public void speak(){
        moveFactor.speak();
    }
}
```  
4、测试profile  
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = App.class)
@ActiveProfiles("dev")  //使用activeProfile设置当前测试环境为dev
public class SpringTest {

    @Autowired
    Person p;

    @Test
    public void testProfile(){
        p.speak();
    }

}
```
如果在web项目中，可以通过xml的方式进行配置当前项目的运行环境，来决定注入相应的bean。  
```xml
<context-param>
  <param-name>spring.profiles.active</param-name>
  <param-value>环境名</param-value>
</context-param>
```
也可以在方法中通过获取ConfigurableApplicationContext对象，使用其中的getEnvironment().get(set)ActiveProfiles("环境名")，来获取或设置当前激活的环境  


## ORM框架(Object-relational mapping)关系映射框架(例如：mybatis、heberntis)  
ORM框架是为了解决面向对象与关系型数据库的不匹配问题。优点是数据访问更抽象、轻便，支持面向对象的封装  

## Spring中bean的自动装配的实现方式  
1、通过xml注解，配置bean标签中autowire属性的值  
```xml
<bean id="employeeDAO" class="com.guor.EmployeeDAOImpl" autowire="byName" />
```
2、通过注解的方式(@Autowired)

## Spring开启基于注解的自动装配（使用@Autowired）  
要使用 @Autowired，需要注册 AutowiredAnnotationBeanPostProcessor，可以有以下两种方式来实现：  
1、在配置文件中的<beans>下引入 <context:annotation-config>  
```xml
<beans>
    <context:annotation-config />
</beans>
```  
2、在配置文件中直接引入AutowiredAnnotationBeanPostProcessor  
```xml
<beans>
    <bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor"/>
</beans>
```

## Spring的@autowaire注解是怎么保证线程安全的  
Spring的@autowaire是通过生成动态代理对象，让线程去操作由ThredLocal修饰的数据副本，从而达到每个线程之间的隔离。  

## Spring中的@Required注解  
@Required注释是为了保证类中所对应的成员属性必须在xml中被初始化配置。直接的理解就是，如果你在某个java类的某个set方法上使用了该注释，那么该set方法对应的属性在xml配置文件中必须被设置，否则就会报bean初始化异常。  
```java
public class Student {
    private String name;

    @Required     //该注释放在的是set方法中，如果没有在xml配置文件中配置相关的属性，就会报异常
    public void setAge(Integer age) {
        this.age = age;
    }
}
```
```xml
    <!-- student bean的定义 -->
    <bean id = "student" class="com.how2java.w3cschool.required.Student">
    <property name = "age" value="5"/>
    </bean>
```

## Spring的@Qualifier注解  
定义了一种类型的多个bean组件，同时在java类的注解中又想把其中一个bean对象作为属性，那么此时可以使用@Qualifier指定@Autowired或@Resource注解加载特定的bean对象。  
```java

public interface IP{
}
@Bean("IP1")
public class IP1 implements IP{

}
@Bean("IP2")
public class IP2 implements IP{

}
//使用@Qualifier指定bean对象加@Autowired进行填充  
@Colltroller
public class Iptest{
    @Autowired
    @Qualifier("IP1")  //Qualifier的意思是合格者，通过这个标示，表明了哪个实现类才是我们所需要的
    public IP ip;
}

//使用@Resource进行填充   
@Colltroller
public class Iptest{
    @Resource(name="IP1")  //使用@resource注入时比较简单,注解自带了“name”的val就是所需的注解名称。
    public IP ip;
}
```

## Spring的七大事务的传播机制  
当我们调用一个基于Spring的Service接口方法（如UserService#addUser()）时，它将运行于Spring管理的事务环境中，Service接口方法可能会在内部调用其它的Service接口方法以共同完成一个完整的业务操作，因此就会产生服务接口方法嵌套调用的情况， <font color='red'>Spring通过事务传播行为控制当前的事务如何传播到被嵌套调用的目标服务接口方法中。</font>
 Spring在TransactionDefinition接口中规定了7种类型的事务传播行为，它们规定了事务方法和事务方法发生嵌套调用时事务如何进行传播：  
 1、PROPAGATION_required：如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。  
 2、PROPAGATION_supports：有事务则在当前事务中执行，如果当前没有事务，就以非事务方式执行。    
 3、PROPAGATION_mandatory：使用当前的事务，如果当前没有事务，就抛出异常。  
 4、PROPAGATION_required_new：请求新建事务，如果当前存在事务，把当前事务挂起。(启动一个新的, 不依赖于环境的 "内部" 事务. 这个事务将被完全 commited 或 rolled back 而不依赖于外部事务, 它拥有自己的隔离范围, 自己的锁。当内部事务开始执行时, 外部事务将被挂起, 内务事务结束时, 外部事务将继续执行.)  
 5、PROPAGATION_not_supports：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。(外部方法的Transaction暂停直至innerMethod执行完毕)  
 6、PROPAGATION_never：以非事务方式执行，如果当前存在事务，则抛出异常。  
 7、PROPAGATION_nested：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则新建一个事务。  

## Spring注入一个集合  
list和set都使用value标签；map使用entry标签；props使用prop标签；  
```java
//props
<props>
    <!-- Properties的value值写在prop标签中 -->
    <prop key="username">root</prop>
    <prop key="password">gaoqize</prop>
</props>
//set  
<set>
    <value>foo</value>
    <value>uoo</value>
</set>
//list
<list>
    <value>foo</value>
    <value>uoo</value>
</list>
//map
<map>
    <entry key="key01" value="张三"></entry>
    <entry key="key02" value="18"></entry>
</map>
```
