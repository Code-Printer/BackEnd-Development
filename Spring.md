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
### 根据bean配置的类的类型获取实例对象  
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



