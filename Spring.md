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


