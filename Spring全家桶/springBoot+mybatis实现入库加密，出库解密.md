# 字段入库加密和出库解密  
## 使用拦截器实现
1、实现Interceptor接口，定义入库和出库拦截器
2、自定义两个注解，一个用在类上，一个用在加密字段上
3、在入库的拦截器方法上，首先判断入库的参数对象有没有被自定义的注解(反射获取参数对象)
4、如果有，再判断类中的那个字段有备注了字段加密注解，有，则使用加密方法对该字段加密（反射获取类中的属性）
5、解密同上
实例：
```
https://blog.csdn.net/xiaohui151211/article/details/127075384
```   
## 使用Spring aop切面实现  
1、在pom文件引入spring aop组件，使用@Aspect注解定义切面类  
2、在切面类中定义使用@PointCut定义切入点  
3、定义controller方法返回后执行的切面方法，在方法上使用@AfterReturn  
4、定义在入库前的切面方法，在方法上注解@Before

参考实例：
```
https://blog.csdn.net/dadawdawdadadw/article/details/118336658
```