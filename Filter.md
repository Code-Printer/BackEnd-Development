## Filter过滤器
### Filter介绍(JavaEE的规范，也就是一个接口)
1、Filter是JavaWeb的三大组件之一，其余组件是：Servlet和Listener监听器。  
2、Filter的作用是拦截请求、过滤响应。  
### Filter过滤器的使用  
1、Filter过滤器的使用步骤：  
(1)、编写一个类实现Filter接口，实现三个方法。  
(2)、实现三个方法。(doFilter()只有执行此方法，才可以访问到拦截路径中的资源，如果该方法不执行则说明被拦截、init()、destroy())。  
(3)、在web.xml中配置Filter拦截路径。  
实例演示：在web目录下，有一个Admin目录，此目录下的所有资源都必须用户登录后才能访问，若没有登陆，则跳转到登录页面。  
分析：用户登录后的登录信息都会存放到Session域中，所以检查用户是否登陆，可以判断Session域中是否包含用户的登陆信息。  
代码演示：
1、创建LoginServlet程序，实现Filter接口中的方法
```java
public class LoginServlet implements Filter {
    @Override
    //拦截请求
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        HttpSession session = httpServletRequest.getSession();
        Object user = session.getAttribute("user");
        //如果等于null，说明用户还没有登录，则不应该回应该请求，应该让用户登录
        if (user == null) {   httpServletRequest.getRequestDispatcher("/login.jsp").forward(servletRequest,servletResponse);
        } else {
            //用户有登录，可以继续请求
            filterChain.doFilter(servletRequest,servletResponse);
        }
    }
    @Override
    public void init(FilterConfig filterConfig) throws ServletException { }
    @Override
    public void destroy() { }
}
```
2、配置xml文件
```xml
<!--  Filter标签用于配置一个Filter过滤器，用法与Servlet标签一致  -->
<filter>
    <filter-name>LoginServlet</filter-name>
    <filter-class>com.qizegao.Filter.LoginServlet</filter-class>
</filter>
<filter-mapping>
    <filter-name>LoginServlet</filter-name>
    <!--  url标签用于配置拦截路径，也就是访问哪些资源需要被拦截  -->
    <!--  /表示工程路径，映射到web目录  -->
    <!--  /*代表指定目录下的所有文件  -->
    <url-pattern>/Admin/*</url-pattern>
</filter-mapping>
```  
结果：直接在浏览器输入：http://localhost:8080/MyTest/Admin/a.jsp，跳转到login.jsp，无法访问a.jsp，只有登录后，session记录到该用户才可以访问a.jsp。  
注意：
1.Filter过滤器也支持注解，在首行加@WebFilter(“拦截路径”)，则无需web.xml文件。  
2.浏览器不能直接访问实现Filter接口的类，只需访问拦截路径，就会自动的触发doFilter方法。   
### Filter过滤器的生命周期  
1、构造函数
2、init方法  
3、doFilter方法  
4、destory()方法  
其中1和2是在工程启动时，就创建Filter过滤器。3是在请求拦截路径到服务器时自动执行，如果请求访问的不是拦截路径则不执行。4是在web工程停止时执行(销毁Filter过滤器)。  
### FilterConfig类  
1、FilterConfig类是Filter过滤器的配置文件类， 每次创建Filter的时候，也会创建一个FilterConfig类，其中包含了Filter的配置信息。  
2.FilterConfig类的作用是获取Filter过滤器的配置信息：
(1) 获取Filter的名称，即web.xml文件中<filter-name>标签的值：filterConfig.getFilterName();
(2) 获取web.xml文件中标签的值(写在filter标签中的<filter-class>，可写多个)，filterConfig.getInitParameter("param-name")如：  
```xml
<filter>
        <filter-name>SysFilter</filter-name>
        <filter-class>filter.SysFilter</filter-class>
        <init-param>
        <param-name>user</param-name>
        <param-value>password</param-value>
    </init-param>
</filter>
```  
3、获取ServletContext对象：filterConfig.getServletContext()  
### FilterChain过滤器链  
多个过滤器一起工作：  
![result](https://static01.imgkr.com/temp/de1b4f9fcd524eb8bad80687143ec3c7.png)  

1、上述两个Filter拦截的资源路径相同，代表一定会执行两个Filter过滤器的doFilter方法，执行顺序按在xml中的配置来，从高到低执行。  
2、如果两个Filter拦截资源不同，且拦截资源符合Filter1，不符合Filter2，则会执行Filter1 的doFilter方法，且执行其中的chain.doFilter方法时，不会去执行Filter2的doFilter方法，如果没有符合的过滤器就直接去访问资源，之后执行Filter1的后置代码(在chain.doFilter之后的均是后置代码)。  
3、如果请求的资源不符合过滤器1和2的拦截路径，两个doFilter方法都不执行  
4、前置代码、chain.doFilter方法、后置代码都在本类重写doFilter方法中。  
### Filter的拦截路径设置
1、精确匹配
```xml
<url-pattern>/具体项目下的路径文件</url-pattern>
```  
2、目录匹配：web工程下的admin目录下的所有文件  
```xml
<url-pattern>/admin/*</url-pattern>
```  
3、后缀名匹配(例jsp的文件)
```xml
<url-pattern>*.jsp</url-pattern>
```  
注意：Filter过滤器只关心请求的地址是否符合拦截路径，不会关心请求的资源是否存在。
## Filter与Interceptor的区别  
Filter是Servlet定义的原生组件，脱离spring框架仍然可以使用
Inteceptor是spring定义的接口，可以使用spring的自动装配功能