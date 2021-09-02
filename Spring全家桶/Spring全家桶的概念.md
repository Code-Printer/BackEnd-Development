# Spring的概念
## Spring创建bean的流程：  
1、通过getBean方法从bean工厂中获取对应bean  
2、如果没有对应bean，则使用createBeanInstance方法通过反射的方式创建bean实例  
3、再使用populateBean方法填充该bean实例属性  
4、使用initializeBean方法执行该bean的初始化，以及通过bean的后置处理器对bean进行增强 
## Spring的三级缓存
1、一级缓存：存放完整的单例对象   
2、二级缓存：存放半成品的bean或代理对象，用于解决循环依赖(简单理解：就是在a类中引用了b类对象，在b类中引用了a类对象。故创建a的bean时，会需要创建b的bean，但填充b的属性时，a的bean是一个半成品，需要放在二级缓存中。)  
3、三级缓存(有一个lambda表达式，是将为未初始化完全或填不完全的bean转化成其代理对象)：存放单例工厂，用于生成动态代理对象，返回该bean的代理对象(简单理解：当当前bean被其他bean引用时，如果自己未实例化完全，是一个半成品bean，则需要使用其动态代理，返回自己的动态代理对象。)    
### 循环依赖问题
Spring框架中在A类中引用B类，在B类中引用A类。此时如果获取A的bean对象时，先会从bean容器中寻找A的bean对象，发现不存在则会使用createBeanInstance创建A的bean对象，创建完成后需要去填充A的bean组件属性，此时发现A中的属性是B的对象，会去bean仓库中寻找B的bean，发现B的bean不存在，就会去创建B的bean，当填充B的属性时，发现B中的属性为A的对象，此时A的bean为一个半成品，故需要使用二级缓存，将半成品A的bean存放在二级缓存中，以便B进行属性填充时使用。  
![result](https://static01.imgkr.com/temp/182e6ed55c974c3fb1fe9a4f15873623.png)
### 动态代理问题   
动态代理是指在一个类中，如果调用另一个类，而这个类并未在bean工厂创建完整的bean，此时需要使用这个另一个类的代理对象。  

## Spring的@autowaire注解是怎么保证线程安全的  
Spring的@autowaire是通过生成动态代理对象，让线程去操作由ThredLocal修饰的共享变量，从而达到每个线程之间的隔离。  

## Spring的自动装配原理  
自动装配：在springboot中使用少量注解和配置即可使用引入的依赖功能  
Spring Boot 通过@EnableAutoConfiguration开启自动装配，再通过SpringFactoriesLoader最终加载META-INF/spring.factories中的自动配置类，实现自动装配。(自动配置类是通过@Conditional按需加载的配置类的)，想要其生效必须引入spring-boot-starter-xxx包实现起步依赖。  


 
