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