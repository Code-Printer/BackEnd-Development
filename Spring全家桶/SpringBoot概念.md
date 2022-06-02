## Springboot  
### SpringBoot与Spring的区别  
SpringBoot与Spring相比简化了配置(可以快速搭建项目),集成了许多框架(避免框架间的版本冲突)  
SpringBoot的缺点：版本迭代快，无需配置导致的出错无法定位。  
### SpringBoot与SpringMVC的区别  
SpringMVC提供一种轻度耦合的方式开发web应用。  
SpringBoot实现了自动配置，无需xml文件手动配置，降低了项目搭建的复杂度。
### SpringBoot的常用注解及作用  
#### @SpringBootApplication
这个注解是Spring Boot最核心的注解，用在Spring Boot的主类上，标识这是一个Spring Boot应用，用来开启Spring Boot的各项能力。实际上这个注解是@Configuration,@EnableAutoConfiguration,@ComponentScan三个注解的组合。  
#### @EnableAutoConfiguration
允许Spring Boot自动配置注解，开启这个注解之后，Spring Boot就能根据当前类路径下的包或者类来配置Spring Bean。  
#### @ComponentScan  
组件扫描。让spring Boot扫描Configuration类并把它加入到程序上下文。@ComponentScan注解默认就会装配标识了@Controller，@Service，@Repository，@Component注解的类到spring容器中。  
#### @Repository  
用于标注数据访问组件，即DAO层组件。使用@Repository注解可以确保DAO或者repositories提供异常转译(捕获异常后，输出另一个异常(作用在于捕获的如果是底层异常，输出该异常并不能准确表达意思，故需要输出高层可以表意的异常))，这个注解修饰的DAO或者repositories类会被ComponetScan发现并配置，不需要为它们在XML文件中配置项。  
#### @Service  
这个注解修饰的业务逻辑层的类会被ComponetScan发现并配置，不需要为它们在XML文件中配置项。  
#### @RestController  
这个注解修饰的控制层的类会被ComponetScan发现并配置。@RestController相当于@Controller+@ResponseBody，可以返回JSON，XML或自定义media Type数据内容到页面，但注解了@RestController不能返回jsp、html视图页面。  
#### @Component  
泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。  
#### @AutoWired(spring定义的注解方式，只支持spring全家桶)  
从Spring容器中把配置好的Bean拿来用，完成自动装配的工作。它可以对类成员变量、方法及构造函数进行标注。当加上（required=false）时，就算找不到bean也不报错。  
#### @Resource(name="name",type="type")(java定义的注解方式，扩展性强)  
与@AutoWired作用相同  
#### @RequestMapping  
 RequestMapping用来设置请求地址映射的注解；用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。  
### SpringBoot开启事务(事务(ACID)：要么都执行，要么都不执行)  
1、在主类(Main)使用注解@EnableTransactionManagement开启事务  
2、在访问数据库的service方法上添加注解@Transactional即可  
### SpringBoot的自动装配原理  
springboot的自动装配就是通过自定义实现的ImportSelector接口，从而导致项目启动时会自动将所有项目META-INF/spring.factories路径下的配置类注入到spring容器中，从而实现了自动装配。   
### @RequestParam与@RequestBody的区别  
@RequestParam后台接收的参数在请求头中，就是请求路径url中，只能用来接收get请求。    
@RequestBody后台接收的参数在请求体中，只能用来接收post请求。  
### 几种服务器端接收前端传过来的参数方法(常用方法是@RequestParam、@RequestBody)
1、直接把表单的参数写在Controller相应的方法的形参中，适用于get方式提交，不适用于post方式提交。  
```java
    /**
     * 直接把表单的参数写在Controller相应的方法的形参中
      * @param username
     * @param password
     * @return
     */
    @RequestMapping("/addUser1")
    public String addUser1(String username,String password) {
        System.out.println("username is:"+username);
        System.out.println("password is:"+password);
        return "demo/index";
    }
```  
url形式：http://localhost:8080/demo/addUser1?username=lixiaoxi&password=111111 提交的参数需要和Controller方法中的入参名称一致。  
2、通过HttpServletRequest接收，post方式和get方式都可以。  
```java
/**
     * 通过HttpServletRequest接收
      * @param request
     * @return
     */
    @RequestMapping("/addUser2")
    public String addUser2(HttpServletRequest request) {
        String username=request.getParameter("username");
        String password=request.getParameter("password");
        System.out.println("username is:"+username);
        System.out.println("password is:"+password);
        return "demo/index";
    }
```
url形式：http://localhost:8080/demo/addUser2，通过从Servlet请求中获取请求的参数。  
3、通过在方法的参数中注解@PathVariable获取请求路径中的参数  
```java
/**
     * 通过@PathVariable获取路径中的参数
      * @param username
     * @param password
     * @return
     */
    @RequestMapping(value="/addUser4/{username}/{password}",method=RequestMethod.GET)
    public String addUser4(@PathVariable String username,@PathVariable String password) {
        System.out.println("username is:"+username);
        System.out.println("password is:"+password);
        return "demo/index";
    }
```
url形式：http://localhost:8080/demo/addUser4/lixiaoxi/111111 ，则自动将URL中模板变量{username}和{password}绑定到通过@PathVariable注解的同名参数上，即入参后username=lixiaoxi、password=111111。  
4、使用@ModelAttribute注解获取前端发起的POST请求的form表单数据  
```jsp
<form action ="<%=request.getContextPath()%>/demo/addUser5" method="post"> 
     用户名: <input type="text" name="username"/><br/>
     密  码: <input type="password" name="password"/><br/>
     <input type="submit" value="提交"/> 
     <input type="reset" value="重置"/> 
</form>
```  
java Controller如下  
```java
/**
     * 使用@ModelAttribute注解获取POST请求的FORM表单数据
      * @param user
     * @return
     */
    @RequestMapping(value="/addUser5",method=RequestMethod.POST)
    public String addUser5(@ModelAttribute("user") UserModel user) {
        System.out.println("username is:"+user.getUsername());
        System.out.println("password is:"+user.getPassword());
        return "demo/index";
    }
``` 
5、用注解@RequestParam绑定请求参数到方法入参,当请求参数username不存在时会有异常发生,可以通过设置属性required=false解决,例如: @RequestParam(value="username", required=false)  
```java
 /**
     * 用注解@RequestParam绑定请求参数到方法入参
      * @param username
     * @param password
     * @return
     */
    @RequestMapping(value="/addUser6",method=RequestMethod.GET)
    public String addUser6(@RequestParam("username") String username,@RequestParam("password") String password) {
        System.out.println("username is:"+username);
        System.out.println("password is:"+password);
        return "demo/index";
    }
```
url形式：http://localhost:8080/demo/addUser6?username=lixiaoxi&password=111111 提交的参数需要和Controller方法中的入参名称一致。
6、用注解@RequestBody绑定请求参数到方法入参  用于POST请求  
```java
/**
     * 7、用注解@Requestbody绑定请求参数到方法入参,UserDTO 这个类为一个实体类，里面定义的属性与URL传过来的属性名一一对应。
      * @param username
     * @param password
     * @return
     */
    @RequestMapping(value="/addUser6",method=RequestMethod.POST)
    public String addUser6(@RequestBody UserDTO userDTO) {
        System.out.println("username is:"+userDTO.getUserName());
        System.out.println("password is:"+userDTO.getPassWord());
        return "demo/index";
    }
```



