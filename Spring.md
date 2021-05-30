# Spring  
## IOC概述  
IOC：(inversion of Control)，控制反转，是一个(类、对象)管理的容器。  
容器:管理所有的组件(类、对象等，程序中表示为bean)，使用类时不需要去new对象实例化，而是由容器创建和管理类，使用组件时直接通过容器获取即可。(直接可以通过IOC的getBean()方法实例化类的对象)  

### 使用IOC容器为组件属性赋值并获取组件实例  
1、创建一个Student类，并写好属性、构造器及set和get方法。  
```java
public class Student {
   private String lastName;
   private Integer age;
   private String gender;
   private String email;
   
   //构造器、getter和setter、toString方法
}
```  
2、导如Spring的依赖包(右键一个lib文件夹，放入一下jar包，并点击add to build path)  
![result](https://static01.imgkr.com/temp/cd87484519a747499dcd3437e5451bc4.png)  

3、创建一个Spring.xml的配置文件，进行配置(Spring的xml配置文件中包括了Spring的IOC容器管理的所有组件)，创建一个xml文件，自命名为ioc.xml，文件内容如下：  
![result](https://static01.imgkr.com/temp/af70c13487264878908fd9188e48ed7a.png)  

```java
<!-- 在容器中注册一个Student对象，容器会自动创建该对象，无需手动去new -->
      <!--
       一个bean标签可以注册一个组件(对象、类等)
       class属性写要注册的组件所在类的全类名
       id属性写这个组件的唯一标识，自定义标识名称
   -->
   
   <bean class="com.qizegao.test.Student" id="Student01">
       
       <!--
          bean标签中使用property标签为Student对象的属性赋值
             每给一个属性赋值就要写一个property标签
          name属性写要赋值的属性名
          value属性写赋予的值
       -->
       
       <property name="lastName" value="周杰伦"></property> 
       <property name="age" value="18"></property>
       <property name="email" value="@qq.com"></property>
       <property name="gender" value="男"></property>
       
   </bean>
   
   <!-- 另一个Student对象 -->
   <bean id="Student02" class="com.qizegao.test.Student">
       <property name="lastName" value="昆凌"></property>
   </bean>
```  
4、测试  
```java
public class IOCTest {
   /**
    * 从容器中拿到这个组件(Student对象)
    */
   @Test
   public void test() {
       //ApplicationContext表示IOC容器的接口   
       //ClassPathXmlApplicationContext:当前应用的xml配置文件的位置
              //根据Spring的配置文件得到IOC容器对象
       ApplicationContext ioc = new ClassPathXmlApplicationContext("ioc.xml");
       
       //容器已经创建好了Student对象
       //使用容器对象的getBean方法获取组件(Student对象)，参数中是bean标签的id属性值
       Student bean = (Student) ioc.getBean("Student01");
       System.out.println(bean); 
//Student [lastName=周杰伦, age=18, gender=男, email=@qq.com]
       
       //再获取一个Student对象
       Student bean2 = (Student) ioc.getBean("Student01");
       System.out.println(bean == bean2); //true,因为容器中的组件是单例设计模式(即该组件只有一个实例对象，多个线程共享该对象)。
   }
}
```  
5、注意：  
(1) src目录是源码包开始的路径，称为类路径的开始  
(2) Spring的容器接管了Student的类：  
(3) new ClassPathXmlApplicationContext：Spring的xml配置文件在src目录下(与包同等级)时使用  
(4)new FileSystemXmlApplicationContext：Spring的xml配置文件在别的位置时使用  
(5) 工程名右键new Source Folder，此文件夹的作用与src目录一致  
(6) 同一个组件在IOC容器中默认是单例的(该对象的组件只有一个实例)，是容器创建好时创建好的，即多次从调用容器中 调用同一个组件得到的都是同一个对象，地址值相同  
(7) 容器中没有所要获取的组件时报错：NoSuchBeanDefinitionException  
(8) 容器中使用property标签时会调用setter方法为属性赋值  
(9) property标签的name属性由getter/setter方法的方法名决定：方法名的set前缀去掉，剩余字符串首字母小写就是name属性值，并不是定义时的属性名  
(10) 在容器中创建组件对象时默认调用的是无参构造器  
### 根据bean配置的类的类型获取实例对象(通过类的类型获取实例对象要求容器内配置的该类组件只有一个，如果有多个则会报异常)  
```java
@Test
   public void test2() {
       ApplicationContext ioc = new ClassPathXmlApplicationContext("ioc.xml");
       Student s1 = ioc.getBean(Student.class); //不用类型转换
       /**
        * 1. getBean方法的参数是Class对象时，只适用于该类在bean中只有配置了一个组件，一个类配置多个组件会报错
        * expected single matching bean but found 2
        *
        * 2. 如果容器中一个类有多个组件时，只能使用参数为id属性值的getBean方法
        *
        * 3. getBean方法的参数是Class对象时，得到的结果不用类型转换
        *
        * 4. getBean方法的参数是Class对象时，会获取到此类对象(如果有子类就获取子类对象)
        */
   }
```  
### 通过构造器给bean组件对应的对象赋值(这里演示了可以不通过name属性赋值，但有顺序要求，所以后面直接还是用name属性更加直观)  
```xml
 <bean id="Student04" class="com.qizegao.test.Student">
       <!-- 调用有参构造器创建对象并为其属性赋值 -->
       <!-- public Student(String lastName, Integer age, String gender, String email) -->
       <!-- 有几个参数写几个 constructor-arg 标签，name属性对应参数名，value属性对应赋予的值 -->
       
       <constructor-arg name="lastName" value="郭富城"></constructor-arg>
       <constructor-arg name="email" value="@qq.com"></constructor-arg>
       <constructor-arg name="age" value="22"></constructor-arg>
       <constructor-arg name="gender" value="男"></constructor-arg>
   </bean>
   <bean id="Student05" class="com.qizegao.test.Student">
       <!--
          1. xml文件的constructor-arg标签 可以不使用name属性，但要求这些标签的顺序必须与构造器中参数的顺序一致
          1. 构造器重载时，可以在 constructor-arg标签 中使用type属性指定参数的类型
          2. 可以在 constructor-arg标签 中使用index属性指定构造器中参数的索引，从0开始
        -->
       
       <!-- public Student(String lastName, String age, String gender, String email) -->
       <!-- public Student(String lastName, Integer age, String gender, String email) -->
       
       <constructor-arg value="郭富城"></constructor-arg>
       <constructor-arg value="22" index="1" type="java.lang.Integer"></constructor-arg>
       <constructor-arg value="男"></constructor-arg>
       <constructor-arg value="@qq.com"></constructor-arg>
   </bean>
```  
### 为类中各种类型的属性赋值  
1、xml中如果没有给属性赋值，则属性使用默认值：引用类型为null值，基本数据类型为对应的默认值，如果定义类时给属性赋予了默认值，则容器中的对象默认值为初始时赋予的。  
2、xml中编写的内容都是文本格式，使用property标签赋值时，会自动的进行类型转换(把文本格式转换为对应的属性的类型)，如：<font color="white">String类型的属性在xml中使用property标签进行赋值：value = “null”，赋予的是字符串null，并不是null值。</font>  
3、给属性赋予特殊值时，应该在property标签中进行赋值，而不是使用value属性赋值。  
<font color="white">例：给类的属性lastName赋予null值：</font>
```xml
<property name="lastName">
     <null /> <!-- 赋予null值的标签  -->
</property>
```  
<font color="white">例：给类类型的属性赋值</font>  
1、属性被赋值的类已经在本容器中有定义组件：则在property标签中使用ref标签在ref标签中绑定bean属性，值为其id值，可以引用容器中已经存在的其他组件。注：ref属性表示的是引用，代表地址值相同
```xml
<bean class="com.company.Student" id="student3">
        <property name="eamil">
        <ref bean = "student1">
        </property>
</bean>
```
2、容器中没有定义要赋值的属性所在类的组件
```xml
<bean id="Student09" class="com.qizegao.test.Student">
   <property name="book">
      <!-- 可以使用bean标签创建Book对象：引用内部bean -->
      <bean class="com.qizegao.test.Book">
         <property name="author" value="桂纶镁"></property>
      </bean>
   </property>
</bean>
<!-- 内部bean不可以被getBean方法获取，只能内部使用，所以一般不写id属性 -->
```  
<font color="white">例：给List集合类型的属性赋值</font>  
```xml
<property name="books"> <!-- books为List集合类型的属性名 -->
      <list> <!-- list标签体中添加每一个元素-->
        
         <!-- 第一个元素 -->
         <bean class="com.qizegao.bean.Book">
             <property name="bookName" value="西游记"></property>
         </bean>
             
         <!-- 第二个元素，引用外部一个元素 -->
         <ref bean="book01" /> <!-- book01是xml中另一个组件对象 -->
     
      </list>
   </property>
```
<font color="white">例：给Map集合类型的属性赋值</font>
```xml
   <property name="maps"> <!-- maps是Map类型的属性名 -->
      <map> <!-- 在map标签体中编写键值对 -->
         <!-- 一个entry代表一个键值对 -->
         <entry key="key01" value="张三"></entry>
         <entry key="key02" value="18"></entry>
         <entry key="key03" value-ref="book01"></entry> <!-- 引用其他组件 -->
         <entry key="key04"> <!-- 使用内部bean -->
             <bean class="com.qizegao.bean.Car">
                <property name="carName" value="宝马"></property>
             </bean>
         </entry>
         <entry key="key05">
             <value>李四</value>
         </entry>
      </map>
   </property>
```   
<font color="white">例：给Properties类型的属性赋值</font>  
```xml
<!-- 为Properties类型的属性赋值，所有的key、value都是String型的 -->
   <!-- Student类中添加属性Properties properties -->
   <bean id="Student10" class="com.qizegao.test.Student">
       <property name="properties">
          <props>
             <!-- Properties的value值写在prop标签中 -->
             <prop key="username">root</prop>
             <prop key="password">gaoqize</prop>
          </props>
       </property>
   </bean>
```   

### 创建各种类型的bean  
<font color="white">1、使用util(防止标签重复)名称空间创建List对象</font>   
名称空间在xml文件中主要是用于防止标签重复。  
1、IDEA中引入名称空间util标签：  

![result](https://static01.imgkr.com/temp/e67c76a27e424889ac810b0a4a417bf2.png)  

代码演示：  
```xml
<!--这是一个bean组件 -->
  <util:list id="util_list" value-type="java.lang.String">
        <value>foo</value>
        <value>uoo</value>
    </util:list>
```  
<font color="white">2、通过继承实现bean配置信息的重用</font>   
```xml
 <!-- parent属性指定当前的配置信息继承于哪个组件 -->
   <!--
       1. 仅仅指的是配置信息的继承，并不代表是类的继承，即二者没有子父类的关系
       2. 相同的配置信息不用写，需要修改的配置信息写出
    -->
   <bean id="Student11" parent="Student01">
       <property name="lastName" value="李刚"></property>
       <!-- 其余三项与Student01相同 -->
   </bean>
```  
<font color="white">3、创建单实例和多实例的bean</font>  
```xml
   <!-- scope属性可指定此bean为单实例或多实例(默认是单实例的)
       1. singleton：单实例(不使用scope属性默认的形式)：
          (1) 创建容器时创建此组件，保存在容器中
          (2) 每次获取的都是已经创建好的组件(同一个)
       2. prototype:多实例：
          (1) 创建容器时不会创建此组件
          (2) 获取此组件时才创建此组件，并且不会保存在容器中
          (3) 每次获取该组件对象的都是一个新的对象
   -->

   <bean id="Student12" class="com.qizegao.test.Student" scope="prototype"></bean>
```  
<font color="white">4、通过实现FactoryBean创建爱你bean</font>  
FactoryBean是Spring的一个接口，此接口中有三个抽象方法，只要是这个接口的实现类，Spring 都认为是一个工厂：
i. Spring会自动调用该工厂的工厂方法创建实例
ii. 该工厂创建的组件(实例)，ioc容器创建时不会创建此组件
iii. 获取组件时才创建组件(对象)，不论FactoryBean的实现类创建的对象是否为单例  
(1) 创建一个实现了FactoryBean的工厂类  

```java
 public class ImplFactoryBean implements FactoryBean<Student>{
 
   //工厂方法，返回创建的对象
   @Override
   public Student getObject() throws Exception {
       Student student = new Student();
       return student;
   }
   
   //返回创建的对象的类型
   //Spring会自动的调用这个方法来确定创建的对象是什么类型
   @Override
   public Class<?> getObjectType() {
       return Student.class;
   }
  
   //创建出的对象是否为单例
   @Override
   public boolean isSingleton() {
       return true;
   }
 }
```  

(2)在xml中创建bean  
```xml
<bean id="factorybeanimpl" class="com.qizegao.test.ImplFactoryBean"></bean>
```  
(3) 测试  
```java
   public void test() {
       ApplicationContext ioc = new ClassPathXmlApplicationContext("ioc.xml");
       Student student1 =  (Student)ioc.getBean("factorybeanimpl");
       Student student2 =  (Student)ioc.getBean("factorybeanimpl");
       System.out.println(student1 == student2); //true
     }
```  
<font color="white">5、创建带有生命周期方法的bean(组件)</font>  
 可以为bean自定义一些生命周期方法，Spring在创建或者销毁bean时会自动调用指定的方法， 这些生命周期方法必须无参，但可抛出任何异常。   
代码演示：  
(1)、在Student类中添加init方法和destory方法  
```java
   //初始化方法
   public void init() {
       System.out.println("执行了初始化方法");
   }
   //销毁方法
   public void destory() {
       System.out.println("执行了销毁方法");
   }
```  
(2)在xml中配置该类的组件  
```xml
 <bean id="Student15" class="com.qizegao.test.Student"
        init-method="init" destroy-method="destory"></bean>
```  
(3)测试  
```java
   public void test1() {
       //此类型有close方法
       ConfigurableApplicationContext ioc = new ClassPathXmlApplicationContext("ioc.xml");
       ioc.close();
       /**
        * 运行结果：
        * 执行了初始化方法
        * 执行了销毁方法
        */
   }
```  
注意：(1) 单例bean(创建容器时就创建单例的bean配置对象)的生命周期：(容器创建)调用构造器 → 调用初始化方法 → (容器关闭)调用 销毁方法   
(2) 多实例bean(获取组件时才创建对应配置的对象)的生命周期：<font color="white">(获取bean)</font>调用构造器 → 调用初始化方法 → (容器关闭)<font color="white"> 不调用bean的销毁方法</font>     

### bean的后置处理器  
Spring有一个接口：BeanPostProcessor，即后置处理器，其中有两个抽象方法，分别可以在bean的 初始化前后自动的调用。  
代码演示：  
1、编写后置处理器(实现BeanPostProcessor接口重写接口中的方法)。  
```java
 public class ImplBeanPostProcessor implements BeanPostProcessor {
  
    /**
    * bean初始化之前调用
    * 参数1：将要初始化的bean
    * 参数2：xml文件中配置的id
    */
   @Override
   public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
       System.out.println("初始化之前调用的方法");
       return bean;
   }

   /**
    * bean初始化之后调用
    * 参数1：初始化的bean
    * 参数2：xml文件中配置的id
    */
   @Override
   public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
       System.out.println("初始化之后调用的方法");
       return bean;
   }
}
```
2、在xml中配置该后置处理器的bean组件  
```xml
<bean id="beanpostprocessor" class="com.qizegao.test.ImplBeanPostProcessor"></bean>
```
3、测试  
```java
public void test1() {
       ConfigurableApplicationContext ioc = new ClassPathXmlApplicationContext("ioc.xml");
       ioc.close();
       /**
        * 运行结果：
        * 初始化之前调用的方法
        * 执行了初始化方法
        * 初始化之后调用的方法
        * 执行了销毁方法
        */
   }
```
注意：(1)初始化之前、之后执行与相应组件有没有初始化方法并无关系。  
(2)xml文件中一旦使用了此实现类的配置语句，此xml文件中所有的组件都会执行初始化之前、 之后的方法。  
(3)单实例后置处理器执行过程：(容器创建)调用构造器 → 初始化之前执行的方法 → 初始化方法 → 初始化之后执行的方法 → (容器销毁)调用销毁方法。  
(4)初始化之前调用的方法返回null，则此组件在初始化之前就已经成为了null，初始化之后调 用的方法返回null，则此组件初始化之后就会成为null。   
## 注解式创建组件对象  
### 使用数据库的查询更新进行spring框架演示  
1、在maven项目中引入mysql、Druid和Spring。  
```xml
   <!--引入@Test -->
   <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
   <!--引入spring-context -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${org.springframework.version}</version>
      <scope>runtime</scope>
    </dependency>

    <!-- spring的持久层依赖 -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-orm</artifactId>
      <version>${org.springframework.version}</version>
    </dependency>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-api</artifactId>
      <version>RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>3.2.8.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.6</version>
    </dependency>

    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.17</version>
    </dependency>
```  
2、spring的ioc容器配置xml中编写,记得引入context名称空间
```xml
<!-- location属性固定写法：classpath表示引用类路径下的资源-->
    <context:property-placeholder location="classpath:/properties.propertity"/>

    <!-- 创建一个数据库连接池 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">

        <!-- ${key}动态取出配置文件中某个key对应的值 -->

        <property name="username" value="${jdbc.username}"></property>
        <property name="password" value="${jdbc.password}"></property>
        <property name="url" value="${jdbc.url}"></property>
        <property name="driverClassName" value="${jdbc.driverClassName}"></property>

    </bean>
```  
3、测试数据库连接  
```java
@Test
    public void test() throws SQLException {
        ApplicationContext ioc = new ClassPathXmlApplicationContext("ioc.xml");
        DataSource dataSource = (DataSource)ioc.getBean("dataSource");
        System.out.println(dataSource.getConnection());
    }
```   
### 通过注解式创建bean对象  
 Spring中有四个注解  
(1) @Controller：推荐给控制层(Servlet)的组件加此注解  
(2) @Service：推荐给业务逻辑层的组件添加此注解  
(3) @Repository：推荐给数据库层(Dao层)的组件添加此注解  
(4) @Component：推荐不属于以上的组件添加此注解  
注意：
i. 给所要创建组件的类添加任何一个注解都可快速的将此组件添加到ioc容器的管理中  
ii. 使用以上四个注解的任何一个均可，但尽量使用推荐  
步骤：  
1、在maven项目中配置Spring aop  
```xml
 <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.8.9</version>
     </dependency>
```  
2、在需要创建对象的类上添加注解
```java
import org.springframework.stereotype.Service;

    @Service
    public class BookService {
       //内容
    }
```  
3、让Spring自动扫描加了注解的组件，在xml文件中使用context名称空间  
```xml
<!-- context:component-scan：自动扫描组件 -->
<!-- base-package：指定扫描的包，会将此包下的所有包的所有加了注解的类，自动的扫描进ioc容器中 -->
<context:component-scan base-package="com.qizegao"></context:component-scan>
<!-- com.qizegao.test 与 com.qizegao.util包都会被扫描，扫描之后符合条件的类图标会加小s -->
```  
注意：
使用注解加入到容器中的组件，和使用配置文件加入到容器中的组件默认行为是一致的(默认组件是单例模式)：  
i. 组件的默认id是类名首字母小写  
ii. 组件默认是单例的  
4、测试  
```java
public void test() {
    ApplicationContext ioc = new ClassPathXmlApplicationContext("ioc.xml");
    System.out.println(ioc.getBean("bookService") == ioc.getBean("bookService"));
    //true
  }
```  
注意：  
i. 可以修改组件的id  
ii. 可以修改组件为多实例，如下： 
```java
@Service("newName") // 修改id为newName
  @Scope(value="prototype") // 修改bean为多实例
  public class BookService {
       //内容
  }
```   
测试  
```java
  public void test() {
       ApplicationContext ioc = new ClassPathXmlApplicationContext("ioc.xml");
       System.out.println(ioc.getBean("newName") == ioc.getBean("newName"));
       //false
   }
```  
### 指定Spring扫描时不包含、只包含的组件  
```xml
//不包含
   <context:component-scan base-package="com.qizegao">
       <!-- 使用context:exclude-filter指定扫描时不包含的组件：
             1. type=annotation：按照注解进行排除，使用了expression中指定的注解(全类名)就不会被包含进来
             2. type=assignable：按照指定的类进行排除，expression中指定要排除的全类名
        -->
       <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service"/>
   </context:component-scan>
//只包含
    //Spring默认是把包中所有满足条件的组件全部扫描进来，故应当先禁用默认的扫描规则
   <!-- use-default-filters="false"：禁用默认的扫描规则 -->
   <context:component-scan base-package="com.qizegao" use-default-filters="false">
        <!-- 使用context:include-filter指定扫描时只包含的组件：用法与context:exclude-filter一致 -->
        <context:include-filter type="assignable" expression="com.qizegao.test.BookService"/>
   </context:component-scan>
```  
### 使用@Autowired注解给在容器中配置组件的类的类型属性赋值：根据类型实现自动装配  
自动装配：自动的为某个属性赋值，一定是去容器中找这个属性对应的组件(以该属性的类型在容器中寻找对应的属性组件)赋值，也就要求此组件 必须在容器中，否则报错。  
代码演示
1、在包下创建两个类，使用注解对两个类进行注释  
```java
@Service("book")
public class BookeService {
    @Autowired
    private BookDao bookDao;
    public void save(){
        System.out.println("要去使用BookDao的save方法");
        bookDao.save();
    }
}


@Repository("bookDao")
public class BookDao {
    public void save(){
        System.out.println("正在使用BookDao的save方法");
    }
}
```  
2、xml中让Spring扫描加了注解的组件  
```xml
<context:component-scan base-package="com.qizegao"></context:component-scan>
```   
3、测试  
```java
 @Test
    public void test() throws SQLException {
        ApplicationContext ioc = new ClassPathXmlApplicationContext("ioc.xml");
        BookeService bookeService = ioc.getBean(BookeService.class);
        bookeService.save();
    }
```
### @Autowired注解自动装配原理   
先按照类型(BookDao)去容器中找对应的组件：getBean(BookDao.class)：  
(1) 找到一个，赋值  
(2) 没有找到，抛异常  
(3) 找到多个(会将此类的子类也找到)：  
将变量名(bookDao)作为id继续去容器中找对应的组件：
i. 匹配成功，装配  ii. 匹配失败，报错   
注意：
(1)@Autowired  
@Qualifier(限定词)(“xxx”) //使用此注解可以使用xxx作为id去匹配，而不是使用变量名(原程序的变量bookDao)作为id;  
(2) 匹配到则装配，匹配不到则赋null值：@Autowired(required=false)  
### 在方法上使用@Autowired  
1、这个方法会在bean创建的时候自动运行  
2、这个方法的每个参数都会按照上述自动注入值(参数的赋值会在ioc的容器中找寻相应组件)  
代码实现：  
1、BookService中的方法  
```java
    //BookService与BookDao均已注册到ioc容器中
    @Service
    public class BookService {
       @Autowired(required=false)
       //参数的位置也可以使用@Qualifier注解
       public void method(@Qualifier("bookDao")BookDao bookDao) {
           System.out.println(bookDao); 
       }
    }
```  
2、测试  
```java  
  @Test
   public void test() {
       ApplicationContext ioc = new ClassPathXmlApplicationContext("ioc.xml");
       //运行后输出：com.qizegao.test.BookDao@2bbaf4f0
   }
```
### @Autowired与@Resource的区别
@Autowired：是Spring定义的注解，功能最强大，但只支持Spring框架  
@Resource：是Java定义的标准，扩展性强，在非spring的容器中也可以使用   

### 泛型依赖的注入  
注册一个组件时，它的泛型也是参考标准(带上泛型去找ioc的相应组件)，如下：    
![result](https://static01.imgkr.com/temp/6f30866c9a514b7d9b0d1cc9669d87f6.png)  

### Spring的单元测试  
1、引入依赖(Spring-test)  
2、在测试类之上使用两个注解：
① @ContextConfiguration(locations=“classpath:ioc.xml”)
指定Spring配置文件的位置  
② @RunWith(SpringJUnit4ClassRunner.class)
指定用哪种驱动进行单元测试，默认是Junit  
(3) 好处：直接在要获取的组件上加注解@Autowired，Spring 会自动装配，无需使用getBean方法获取组件。
(4) 使用案例：  
```java
 @ContextConfiguration(locations="classpath:ioc.xml")
    @RunWith(SpringJUnit4ClassRunner.class)
    public class test {
   
       @Autowired
       BookDao bookDao;
   
       @Test
       public void test() {
           System.out.println(bookDao);
           //com.qizegao.test.BookDao@62e136d3
       }
    }
```  

## Spring AOP动态代理的两种实现方法(基于接口的JDK动态代理和基于没有接口的CGLIB动态代理:区别在于代理类实现的是接口还是继承的类)   
SpringAOP声明的两种实现方式：1、基于xml的声明实现方式(配置文件xml中配置pointcut, 在java中用编写实际的aspect 类, 针对对切入点进行相关的业务处理。)2、基于注解的声明方式(将写在spring 配置文件中的连接点写到注解里面)。   
AOP：面向切面编程，将段代码动态的切入到指定方法的指定位置。动态的切入的好处是不写死在业务逻辑方法中，修改方便。  
AOP的底层原理是动态代理，而且并没有要求被代理类一定实现某一接口(言外之意是被代理类也可以是继承自某个类)。   
### AOP术语  
![result](https://static01.imgkr.com/temp/cda19e2d83aa4344a81c0635750588dd.png)  

Calculator类中有四个方法，每个方法都有开始、返回、异常、结束四个状态(位置)。  
目标方法：Calculator类中的四个方法(被切入的方法)。 
目标类：包含目标方法的类(Calculator类)  
通知方法：将要切入到目标方法指定位置的方法(在目标方法指定位置执行)。  
切面类：包含通知方法的类。  
横切关注点：所有目标方法的同一位置。  
连接点：某个目标方法的四个位置(开始-结束-异常-返回)都是连接点。  
切入点：通知方法具体要切入的位置。  
切入点表达式：根据这个表达式从连接点中选出切入点。
### AOP使用步骤及实例  
1、在maven中配置  
```xml
<dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.8.9</version>
</dependency>
```  
2、写配置(主要是自动注解配置)  
将目标类(被代理类)和切面类加入到ioc容器中：使用四个注解之一，并在xml中扫描(开启自动组件扫描)：  
(2) 告诉Spring哪一个是切面类(在类上加注解@Aspect)  
(3) 告诉Spring切面类中的每个通知方法都是何时何地运  行：  
① 在通知方法之上加注解告知何时运行：  
![result](https://static01.imgkr.com/temp/145ffd09388a4bee9b8cc63f5b7ddc89.png)   

② 在上面的注解之后加括号，其中写切入点表达式，告知何地运行：( @After("execution(public int com.mrgao.MyCalculator.*(int, int))"))  
切入点表达式的写法：
execution(访问权限符 返回值类型 目标方法的全类名(参数列表))
切入点表达式中通配符、逻辑运算符的使用：  
![result](https://static01.imgkr.com/temp/65173c8438b646e085d8cd8b1464bc30.png)  

3、ioc.xml中配置基于注解的aop模式  
```xml
   <!--  开启基于注解的AOP功能；aop名称空间-->
   <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
   <!-- 配置正确后，通知方法、目标方法之前会有小箭头  -->
```  
4、代码演示  
1、定义接口，后序被代理对象实现(也可以不定义)
```java
public interface Calculator {
   
   public int add(int i, int j);
   public int sub(int i, int j);
   public int mul(int i, int j);
   public int div(int i, int j);
}
```  
2、目标类(被代理的类)  
```java
@Component
//目标类
public class MyMathCalculator implements Calculator{
   @Override
   public int add(int i, int j) {
       return i + j;
   }
   @Override
   public int sub(int i, int j) {
       return i - j;
   }
   @Override
   public int mul(int i, int j) {
       return i * j;
   }
   @Override
   public int div(int i, int j) {
       return i / j;
   }
}
```  
3、切面类(内含通知方法)  
```java
@Component
@Aspect
//切面类
public class LogUtils {
   //以下为通知方法
   
   //在执行目标方法之前运行
   @Before("execution(public int com.qizegao.aop.MyMathCalculator.*(int, int))")
   public static void start() {
       System.out.println("方法开始了！");
   }
   
   //目标方法正常执行完之后执行
   @AfterReturning("execution(public int com.qizegao.aop.MyMathCalculator.*(int, int))")
   public static void afterreturn() {
       System.out.println("正常执行完成！");
   }
   
   //目标方法出现异常之后执行
   @AfterThrowing("execution(public int com.qizegao.aop.MyMathCalculator.*(int, int))")
   public static void afterexception() {
       System.out.println("出现异常啦！");
   }
   
   //目标方法运行结束之后执行
   @After("execution(public int com.qizegao.aop.MyMathCalculator.*(int, int))")
   public static void after() {
       System.out.println("整个方法执行完毕！");
   }
}
```  
4、测试1(通过代理类实现接口的方式即javaJDK的动态代理方式实现代理对象)和2(通过直接继承类的方式，创建代理对象，即CGLIB的方式实现代理)  
```java
 @Test
   public void test1() {
       
       //AOP底层是动态代理，ioc容器中保存的是目标类的代理对象(该代理对象的类型与被代理对象相同)
       //所以使用类型获取代理对象时不能使用MyMathCalculator.class(被代理对象的类型)获取(因为此时容器有两个类型相同的组件，通过类型获取会报错)
       //应该使用二者唯一可以产生联系的实现的接口获取
       
       ApplicationContext ioc = new ClassPathXmlApplicationContext("ioc.xml");
       Calculator calculator = ioc.getBean(Calculator.class);
       calculator.add(1, 2);
       /**
        * 方法开始了！
        * 整个方法执行完毕！
        * 正常执行完成！
        */
       System.out.println(calculator); //com.qizegao.aop.MyMathCalculator@47eaca72
       System.out.println(calculator.getClass()); //class com.sun.proxy.$Proxy11
       //$Proxy是代理对象的类型
   }     

   @Test
   //通过id获取目标类的代理对象
   public void test2() {
       //代理对象的id默认是目标类的类名首字母小写
       Calculator calculator = (Calculator)ioc.getBean("myMathCalculator");
       System.out.println(calculator.getClass()); //class com.sun.proxy.$Proxy11
   }
```
2、即使被代理对象(目标类)没有实现任何接口，Spring也可为其创建对象，演示如下：  
```java
 @Test
   public void test1() {   
       //如果没有实现任何接口，通过类型获取代理对象时使用的就是本类类型
       MyMathCalculator mymathCalculator =  ioc.getBean(MyMathCalculator.class);
       mymathCalculator.add(1, 2);
       /**
        * 方法开始了！
        * 整个方法执行完毕！
        * 正常执行完成！
        */
       System.out.println(mymathCalculator.getClass());
       //class com.qizegao.aop.MyMathCalculator$$EnhancerByCGLIB$$acfb5ca0
       //没有实现接口时获取到的代理对象是由CGLIB(一个组织)创建
   }

   @Test
   public void test2() {
       
       //如果没有实现任何接口，通过id获取代理对象时使用的就是本类类型对应的id值
       MyMathCalculator myMathCalculator = (MyMathCalculator)ioc.getBean("myMathCalculator");
       System.out.println(myMathCalculator.getClass());
       //class com.qizegao.aop.MyMathCalculator$$EnhancerByCGLIB$$acfb5ca0
   }
```  
5、通知方法的执行顺序(名词字数逐渐变多)  
![result](https://static01.imgkr.com/temp/64c1f34c92d1487ab53d2a5068433339.png)  

### 使用JointPoint获取目标方法的详细信息  
1、在切面类的通知方法  
```java
  /**
    * 可以在通知方法中获取目标方法的详细信息
    * 只需要在通知方法的参数列表上添加一个JoinPoint类型的参数
    * JoinPoint：封装了当前目标方法的详细信息
    */   
   //在执行目标方法之前运行
   @Before("execution(public int com.qizegao.aop.MyMathCalculator.*(int, int))")
   public static void start(JoinPoint joinPoint) {
       //获取到目标方法使用的参数
       Object[] args = joinPoint.getArgs();
       System.out.println("目标方法使用的参数是：" + Arrays.asList(args));
       //获取到目标方法的签名
       org.aspectj.lang.Signature signature = joinPoint.getSignature();
       //获取目标方法的方法名
       String name = signature.getName();
       System.out.println("目标方法的方法名是：" + name);
   }
```  
2、测试  
```java
@Test
   public void test() {
       MyMathCalculator bean = ioc.getBean(MyMathCalculator.class);
       bean.add(1, 2);
       /**
        * 目标方法使用的参数是：[1, 2]
        * 目标方法的方法名是：add
        */
   }
```  
### 获取目标方法的返回值  
1、在切面类的方法中  
```java
 /**
    * 获取目标方法的返回值：
    * 在通知方法的参数中加返回值类型的参数
    * 在注解中说明此参数是用来做什么的(只有在@AfterReturning注解中才有returning)：
    * returning = "参数名"
    * 表示此参数是用来接收目标方法的返回值的
    *
    * 注：切入点表达式是注解中的value属性值
    */
   
   //目标方法正常执行完之后执行
   @AfterReturning(value="execution(public int com.qizegao.aop.MyMathCalculator.*(int, int))", returning="obj")
   public static void afterreturn(JoinPoint joinPoint, Object obj) {
       System.out.println("目标方法的参数是：" + Arrays.asList(joinPoint.getArgs()));
       System.out.println("目标方法的返回值是：" + obj);
   }
```  
2、测试  
```java
@Test
   public void test() {
       MyMathCalculator bean = ioc.getBean(MyMathCalculator.class);
       bean.add(1, 2);
       /**
        * 目标方法开始执行了
        * 整个方法执行完毕！
        * 目标方法的参数是：[1, 2]
        * 目标方法的返回值是：3
        */
   }
```  
### 获取目标方法的异常信息  
用法与获取目标方法的返回值类似，只是只有在@AfterThrowing注解中才能使用：
throwing = “异常类型的参数名”  
### 可重用的切入点表达式(在目标类的那个目标方法进行切入)  
可重用的切入点表达式：修改切入点表达式时需要去每个通知方法的切入点表达式一个一个的去修改，如果将这些通知方法的切入点表达式抽取出来，别的通知方法都来引用，则修改一处即可使所有通知方法的切入点表达式都修改。  
1、使用步骤：    
(1) 随便声明一个方法体中没有任何内容的返回值为void的空方法   
(2) 给声明的此方法加注解@Pointcut(“抽取的切入点表达式”)(@Pointcut("execution(public int com.mrgao.MyCalculator.*(int, int))"))  
(3) 在其余通知方法原来使用切入点表达式的地方修改为此方法名  
2、代码演示：  
```java
    @Pointcut("execution(public int com.mrgao.MyCalculator.*(int, int))")
    public void myPointCut(){};

    @Before("myPointCut()")
    public void start(JoinPoint joinPoint){
        Object[] arg = joinPoint.getArgs();
        System.out.println(Arrays.asList(arg));
        Signature signature = joinPoint.getSignature();
        System.out.println(signature.getName());
    }
```    
### 环绕通知  
以上四种通知方法结合在一起就是环绕通知。  
环绕通知的通知方法中有一个ProceedingJoinPoint类型的参数，ProceedingJoinPoint接口继承于 JoinPoint接口，有一个返回Object类型的proceed(Object[] args)方法，相当于动态代理中的 method.invoke方法，用来执行目标方法，参数是目标方法的参数列表。  
代码演示:  
```java
 @Around("execution(public int com.qizegao.aop.MyMathCalculator.add(int, int))")
   public Object myAround(ProceedingJoinPoint pjp) throws Throwable {
       Object[] args = pjp.getArgs();
       Object proceed = null; //记录目标方法的返回值
       try {
          System.out.println("环绕通知的@Before执行");
          
          proceed = pjp.proceed(args); //method.invoke();
          
          System.out.println("环绕通知的@AfterReturning执行");
       } catch (Exception e) {
          System.out.println("环绕通知的@AfterThrowing执行");
       } finally {
          System.out.println("环绕通知的@After执行");
       }
       System.out.println("目标方法的返回值是:" + proceed);
       return proceed; //返回目标方法的返回值
   }

   @Test
   public void test() {
       MyMathCalculator bean = ioc.getBean(MyMathCalculator.class);
       bean.add(1, 2);
   }
```
### 五种方法都是用的执行顺序  
![result](https://static01.imgkr.com/temp/5e35f08170ec4e0fae5a1e8dd4e6dec8.png) 

由于环绕通知的@AfterThrowing会先执行，故出现异常如果环绕通知将其catch掉，则普通的@AfterThrowing将检测不到这个异常，故需要在环绕通知方法的@AfterThrowing中将此异常抛出去，方便普通@AfterThrowing也可检测到此异常
可以看出环绕通知比其余通知先执行  
### 环绕通知与其余四个通知的不同  
1、环绕通知比其他通知先执行。
2、其余四个通知都是简单的通知方法，不可以对目标方法进行修改  
3、环绕通知可以修改目标方法的参数(获取到之后可以在通知方法中修改)，也可以修改目标方法的 返回值(直接修改通知方法中return语句的返回值)    
### 多个切面类的运行顺序  
使用@Order注解指定切面类的运行顺序，注解中是int型的数字，数字越小越先执行；如果没有使用@Order注解，则切面类的执行顺序由类名按字母排序的大小决定  
代码演示：
```java
@Component
@Aspect
@Order(1)
public class class1 {
   //在执行目标方法之前运行
   @Before("execution(public int com.qizegao.aop.MyMathCalculator.*(int, int))")
   public static void start() {
       System.out.println("class1的普通@Before执行");
   }
   
   //目标方法正常执行完之后执行
   @AfterReturning("execution(public int com.qizegao.aop.MyMathCalculator.*(int, int))")
   public static void afterreturn() {
       System.out.println("class1的普通@AfterReturning / @AfterThrowing执行");
   }
   
   //目标方法出现异常之后执行
   @AfterThrowing("execution(public int com.qizegao.aop.MyMathCalculator.*(int, int))")
   public static void afterexception() {
       System.out.println("class1的普通@AfterThrowing执行");
   }
}


@Aspect
@Component
@Order(2)
public class class2 {
   //在执行目标方法之前运行
   @Before("execution(public int com.qizegao.aop.MyMathCalculator.*(int, int))")
   public static void start() {
       System.out.println("class2的普通@Before执行");
   }
   
   //目标方法正常执行完之后执行
   @AfterReturning("execution(public int com.qizegao.aop.MyMathCalculator.*(int, int))")
   public static void afterreturn() {
       System.out.println("class2的普通@AfterReturning / @AfterThrowing执行");
   }
   
   //目标方法出现异常之后执行
   @AfterThrowing("execution(public int com.qizegao.aop.MyMathCalculator.*(int, int))")
   public static void afterexception() {
       System.out.println("class2的普通@AfterThrowing执行");
   }
   
   //目标方法运行结束之后执行
   @After("execution(public int com.qizegao.aop.MyMathCalculator.*(int, int))")
   public static void after() {
       System.out.println("class2的普通@After执行");
   }
}
```  
测试：  
![result](https://static01.imgkr.com/temp/2fac868b898b46aa90201c18e4684322.png)  

原理：  
![result](https://static01.imgkr.com/temp/bcb9b5f75688455f8f34002561cbb166.png)  

在切面类加上环绕通知后的执行顺序：  
![result](https://static01.imgkr.com/temp/1d2662b11e0b4afe8749bf5ac2f1048d.png)  









